+++
title = "Cortando o Custo de CI em ~70% Com Runners Self-Hosted em Spot"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-05-19"
categories = [
    "AWS",
]
menu = "main"
+++

### "Por que a conta de CI está maior que a de produção?"

Foi essa pergunta que começou tudo. Os runners hospedados no GitHub são ótimos até você olhar a fatura. Eles cobram por minuto, você não escolhe o hardware, e quando você tem alguns repos movimentados os números deixam de ser fofos.

Cansei de pagar caro por CI. Então movi os jobs mais pesados para meus próprios runners self-hosted, rodando em instâncias spot do AWS EC2, tudo provisionado com Terraform. O resultado foi cerca de 70% a menos nessa linha da conta. Aqui está como fiz, e as partes que ninguém te avisa.

![Topologia do runner self-hosted: um job do GitHub Actions roda numa instância spot EC2 dentro de uma VPC provisionada com Terraform, acessada via SSM em vez de SSH](/images/21-spot-runners.pt.svg)

### Para onde o dinheiro vai de verdade

Os runners Linux do GitHub cobram por minuto. O runner padrão de 2 núcleos é barato por minuto, mas os minutos se acumulam rápido: cada push, cada PR, cada job de matriz, cada retry de teste instável. Multiplique isso por um time inteiro e por um ano e vira um número de verdade.

Duas alavancas resolvem a maior parte:

1. Rodar o trabalho em hardware que você aluga direto da AWS, e não por minuto via intermediário.
2. Alugar esse hardware em spot, e em ARM.

Instâncias spot são capacidade ociosa do EC2 com um desconto grande (frequentemente uns 70% sobre o on-demand). ARM (Graviton, a família `t4g`) é mais barato que x86 para o mesmo trabalho. Combine os dois e a matemática fica divertida.

### Uma pequena comparação de custo

Os números são ilustrativos, mas razoáveis. Digamos que você gaste 50.000 minutos de CI por mês.

```
Opção                           | Custo unitário   | Mensal (~50k min)
--------------------------------|------------------|-------------------
GitHub-hosted (2 núcleos Linux) | $0.008 / min     | ~$400
Frota EC2 t4g.small on-demand   | $0.0168 / hr     | ~$120
Frota EC2 t4g.small SPOT        | ~$0.005 / hr     | ~$36
EC2 t4g.nano SPOT (jobs leves)  | ~$0.0015 / hr    | ~$11
```

O modelo por minuto te pune por estar ocupado. Uma frota spot pequena sempre ligada não. Mesmo depois de somar as coisas chatas (um NAT ou uma subnet pública, CloudWatch Logs, um pouco de transferência de dados) você fica bem abaixo da conta hospedada. É daí que vêm os ~70%.

### O Terraform que faz isso

As partes interessantes são o bloco de spot e o user-data que registra o runner no boot. Todo o resto (VPC, subnets, security groups, IAM) é encanamento padrão, mantido com o mínimo de privilégio.

```hcl
resource "aws_launch_template" "runner" {
  image_id      = data.aws_ami.al2023_arm64.id   # ARM64 / Graviton
  instance_type = "t4g.nano"                      # tiny and cheap; size up per workload

  iam_instance_profile { name = aws_iam_instance_profile.runner.name }
  vpc_security_group_ids = [aws_security_group.runner.id]

  # Spot as "persistent" + "stop": an interrupted runner is stopped and brought
  # back, not terminated and lost. Gate it behind a variable so you can flip to
  # on-demand when you actually need guaranteed capacity.
  dynamic "instance_market_options" {
    for_each = var.use_spot ? [1] : []
    content {
      market_type = "spot"
      spot_options {
        spot_instance_type             = "persistent"
        instance_interruption_behavior = "stop"
      }
    }
  }

  user_data = base64encode(templatefile("${path.module}/user-data.sh.tftpl", {
    repo_url  = var.repo_url                              # owner/repo
    token_arn = aws_secretsmanager_secret.runner_pat.arn # the PAT lives here, not in the AMI
  }))
}
```

O user-data lê o token de registro do Secrets Manager (nunca embutido na AMI) e roda a imagem oficial do runner como um container que o systemd mantém vivo:

```bash
#!/bin/bash
set -euo pipefail

# Pull the registration PAT from Secrets Manager at boot, never from the AMI
ACCESS_TOKEN=$(aws secretsmanager get-secret-value \
  --secret-id "${token_arn}" --query SecretString --output text)

# Run the official runner image. A systemd unit (Restart=always) supervises the
# container, so the runner re-registers and comes back on its own after a reboot
# or a spot stop/start.
docker run --name gha-runner \
  -e REPO_URL="https://github.com/${repo_url}" \
  -e ACCESS_TOKEN="$ACCESS_TOKEN" \
  -e RUNNER_SCOPE="repo" \
  -e LABELS="self-hosted,linux,arm64,spot" \
  myoung34/github-runner:latest
```

O runner é escopado a um único repo (`RUNNER_SCOPE=repo`), não à org inteira, e roda dentro de um container, então um job problemático fica encaixotado. Quer estado limpo a cada job? A imagem também suporta um modo efêmero.

### Acesso ao shell sem chaves SSH

Eu não coloco chaves SSH nessas máquinas. Nunca. A role do IAM dá à instância as permissões do agente SSM, e eu pego um shell pelo Systems Manager Session Manager:

```bash
aws ssm start-session --target i-0abc123def456
```

Sem porta 22 aberta, sem key pairs para rotacionar, sem bastion. Cada sessão fica logada no CloudTrail. O security group tem zero regras de entrada. Só isso já vale a migração.

### Sobrevivendo às interrupções de spot

Spot significa que a AWS pode retomar a instância com dois minutos de aviso. Eu não brigo com isso usando um poller chique. Duas escolhas sem graça fazem o trabalho:

- A requisição spot é **persistent** com o comportamento de interrupção em **stop**, não terminate. Quando a capacidade libera, a mesma instância volta a ligar em vez de sumir de vez.
- O container do runner é supervisionado pelo **systemd com `Restart=always`**, então assim que a máquina volta ele se re-registra e começa a puxar jobs sozinho.

Um build que estava no meio do caminho quando a instância foi retomada é descartado e roda de novo. Para CI isso quase sempre está de bom tamanho, e é bem menos maquinário do que um Auto Scaling group e um poller de metadados cuidando de cada nó. Os logs do runner vão para o CloudWatch, então quando algo quebra às 2h da manhã você tem um rastro em vez de um dar de ombros.

### A parte honesta: o que você agora carrega

A economia é real. Os trade-offs também. Self-hosting de CI significa que você assinou embaixo de:

- **Patching.** Aquela AMI agora é sua. Updates de SO, do agente, do binário do runner, tudo por sua conta.
- **Segurança do runner.** Tudo o que seus jobs podem fazer, o runner pode fazer. Mantenha efêmero, mantenha a role do IAM apertada, mantenha segredos fora da AMI.
- **Interrupções de spot.** Quase invisíveis se você projetar para isso, ocasionalmente irritantes se não.
- **Estado e operação.** Mantenha seu state do Terraform seguro (backend S3 com versionamento, ou no mínimo backups de verdade). A infra agora é sua.

E uma regra dura: **nunca rode runners self-hosted em repos públicos.** O pull request de um estranho pode executar código arbitrário no seu runner. Isso é um tiro no pé apontado direto para sua conta AWS. Self-hosted é só para repos privados e internos da org. Sem exceções.

### Vale a pena?

Para um monorepo privado movimentado ou um time se afogando em builds de matriz, com certeza. Uns 70% a menos de CI, hardware sob seu controle, e zero chaves SSH para cuidar. Para um projeto paralelo minúsculo com três pushes por semana, fique nos runners hospedados e guarde seus fins de semana.

Parei de pagar caro por CI. O Terraform levou uma tarde. De nada.
