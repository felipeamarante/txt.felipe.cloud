+++
title = "Orquestração de Containers na AWS em 2025: ECS vs EKS - Além do Básico"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-04-23"
categories = [
    "AWS",
]
menu = "main"
+++

### "Estamos migrando para containers, mas devemos usar ECS ou EKS?"

Essa pergunta me fizeram pelo menos cem vezes no último ano. Em 2025, com os dois serviços bem maduros, a resposta não é tão simples quanto alguns posts de blog sugerem.

Depois de ajudar dezenas de empresas a tomar essa decisão (e às vezes revertê-la), tenho uma perspectiva mais detalhada do que o usual "ECS é mais simples, EKS é mais poderoso".

### O estado dos serviços de containers da AWS em 2025

Os dois evoluíram bastante:

**Amazon ECS (Elastic Container Service)**
- Service discovery e integração com mesh aprimorados
- Capacidades de agendamento aprimoradas
- Integração mais profunda com o ecossistema serverless da AWS
- Configuração de rede simplificada

**Amazon EKS (Elastic Kubernetes Service)**
- Gerenciamento muito menos doloroso que alguns anos atrás
- Integrações nativas com serviços AWS amadureceram
- Menos overhead operacional com EKS Anywhere e EKS Distro
- Melhores ferramentas de otimização de custos

A diferença entre eles diminuiu bastante, o que paradoxalmente torna a escolha mais difícil.

### Além do marketing: no que cada serviço é REALMENTE bom

#### Pontos fortes do ECS

1. **Integração nativa com AWS**
   O ECS ainda ganha fácil na profundidade de integração com outros serviços AWS. Só a integração com CloudWatch economiza horas de trabalho operacional comparado ao EKS.

2. **Simplicidade operacional**
   Apesar das melhorias no EKS, o ECS requer bem menos conhecimento operacional e tem menos partes móveis.

3. **Custo para implantações menores**
   Para cargas de trabalho menores, o ECS fica tipicamente 15-30% mais barato quando você soma tudo.

4. **Fargate**
   O Fargate no ECS está mais maduro e completo em recursos do que o Fargate no EKS.

#### Pontos fortes do EKS

1. **Portabilidade de cargas de trabalho**
   Implantações multi-cloud e híbridas de verdade funcionam com o EKS, especialmente com o EKS Anywhere e compatibilidade com GKE (Google Kubernetes Engine), AKS (Azure Kubernetes Service) e DOKS (DigitalOcean Kubernetes). As cargas de trabalho rodam de forma consistente independente de qual nuvem está por baixo.

2. **Ecossistema e ferramentas**
   O ecossistema Kubernetes continua bem maior do que o disponível para ferramentas específicas do ECS.

3. **Agendamento e posicionamento avançados**
   Agendamento complexo, regras de afinidade/anti-afinidade e acesso a hardware especializado são mais sofisticados.

4. **Habilidades transferíveis**
   Kubernetes roda em qualquer nuvem. ECS é AWS. Simples assim.

### Framework de decisão: os seis fatores que realmente importam

Depois de dezenas de projetos de migração, estes seis fatores determinam qual serviço faz mais sentido:

#### 1. Expertise da equipe em Kubernetes

```
Baixa expertise → ECS
Alta expertise → EKS
```

Kubernetes ainda tem uma curva de aprendizado real. Se sua equipe já conhece bem o Kubernetes, esse fator empurra fortemente para o EKS. Caso contrário, o investimento em treinamento precisa ser justificado por outros fatores.

#### 2. Estratégia multi-cloud

```
Estratégia apenas AWS → ECS
Estratégia multi-cloud → EKS
```

Se você está comprometido com a AWS a longo prazo, a integração profunda do ECS é uma vantagem real. Se precisa diversificar provedores ou manter capacidades on-premises, o EKS oferece melhor portabilidade. Com o EKS, as mesmas ferramentas e processos funcionam na AWS, GCP (GKE), Azure (AKS) e outros provedores Kubernetes.

#### 3. Recursos operacionais

```
Recursos DevOps limitados → ECS
Equipe DevOps forte → Ambos funcionam bem
```

Em 2025, o EKS ainda exige mais overhead operacional apesar das melhorias da AWS. Organizações com DevOps enxuto costumam lutar com a complexidade do EKS.

#### 4. Complexidade da aplicação

```
Apps containerizados simples → ECS
Requisitos de implantação complexos → EKS
```

Aplicações com padrões de implantação simples ficam bem no ECS. Quando você precisa de implantações canário, regras complexas de afinidade ou agendamento especializado, o EKS tem mais recursos nativos.

#### 5. Requisitos de escala

```
Escala pequena a média → Ambos funcionam bem
Escala massiva (1000+ containers) → Vantagem para o EKS
```

Em escala muito grande, as capacidades de agendamento do Kubernetes dão uma vantagem, mas o ECS reduziu bastante essa diferença.

#### 6. Profundidade de integração com serviços AWS

```
Forte integração com serviços AWS → ECS
Dependências mínimas de serviços AWS → Ambos funcionam bem
```

Se sua arquitetura usa muitos serviços AWS, as integrações nativas do ECS reduzem o código de integração que você precisa manter.

### Casos reais: quem escolheu o quê e por quê

Algumas empresas reais (nomes alterados) que tomaram essas decisões no último ano:

#### Caso 1: Startup FinTech (escolheu ECS)

Equipe de 30 engenheiros, pouca experiência com Kubernetes, fortemente integrada com serviços AWS e com recursos DevOps limitados. Os fatores decisivos foram a simplicidade operacional, integração com AWS e custo.

Resultado: mais de 50 microsserviços no ECS com Fargate, 99,99% de uptime com apenas um engenheiro DevOps em tempo parcial. Estimaram $180.000 de economia em custos operacionais anuais comparado a uma implantação EKS.

#### Caso 2: Provedor de SaaS Empresarial (escolheu EKS)

Mais de 200 engenheiros em várias equipes, experiência existente em Kubernetes, estratégia multi-cloud e requisitos complexos de implantação. O que decidiu: portabilidade entre AWS e GCP, padrões avançados de implantação e o investimento já feito em ferramentas Kubernetes.

Resultado: plataforma rodando na AWS (EKS), GCP (GKE) e Azure (AKS) com manifestos e ferramentas consistentes. Tempo de lançamento de novos recursos caiu 40%. Os mesmos pipelines de CI/CD funcionando nas três nuvens foi o que mais valorizaram.

#### Caso 3: Plataforma de E-commerce (migrou de EKS para ECS)

Equipe de 50 engenheiros que escolheu EKS baseada no hype, lutou com a complexidade do Kubernetes e percebeu que não tinha requisito real de multi-cloud.

Resultado: após migrar para o ECS, incidentes operacionais caíram 60% e custos de infraestrutura de containers caíram 25%. A equipe ficou mais satisfeita e entregou recursos mais rápido.

#### Caso 4: Empresa de análise de saúde (migrou de ECS para EKS)

Equipe de 80 pessoas entre ciência de dados e engenharia, que cresceu além das capacidades do ECS e precisava de agendamento especializado para cargas de trabalho de ML. Além disso, foi adquirida por uma empresa maior que já padronizou em Kubernetes.

Resultado: migrou para o EKS, passou a usar instâncias GPU de forma mais eficiente e padronizou com a infraestrutura da empresa-mãe. A queda de produtividade durante a migração foi recuperada em um trimestre.

### Os fatores que ninguém comenta: o lado humano do ECS vs EKS

Além das comparações técnicas, estes fatores "soft" costumam determinar o sucesso:

#### 1. O entusiasmo da equipe importa

Engenheiros motivados são mais produtivos. Se sua equipe quer usar Kubernetes, forçar o ECS pode custar mais em moral e produtividade do que a diferença de custo operacional.

#### 2. Implicações para recrutamento

Sua escolha afeta quem você consegue contratar:

```
ECS: Pool menor de candidatos, mas focados em AWS
EKS: Pool maior de candidatos com experiência em Kubernetes
```

#### 3. A armadilha do "desenvolvimento orientado ao currículo"

Cuidado com escolher Kubernetes porque os engenheiros querem isso no currículo. Acontece com mais frequência do que as empresas admitem.

#### 4. O custo real das operações

O EKS tipicamente precisa de 1,5-2x mais recursos operacionais que o ECS, mesmo em 2025. Esse custo raramente aparece nas comparações iniciais.

### Comparação de custos: os números que ninguém mostra

A maioria das comparações foca só nos custos diretos do serviço. O quadro real é mais amplo:

```
Fator de Custo                  | ECS                        | EKS
--------------------------------|----------------------------|---------------------------
Custo do serviço                | $0 pelo serviço em si      | $73 por cluster por mês
Gerenciamento do plano de controle | Gerenciado pela AWS     | Parcialmente gerenciado (ainda requer atualizações)
Ferramentas de monitoramento    | Incluídas no CloudWatch    | Requer configuração adicional
Pessoal operacional             | Menor                      | Maior
Investimento em treinamento     | Menor                      | Maior
```

Para uma implantação típica de médio porte (50 serviços, ~200 containers):

```
Custo anual total do ECS: ~$120.000
Custo anual total do EKS: ~$160.000
```

A diferença vem principalmente dos custos operacionais, não das cobranças diretas da AWS.

### Comparação de desempenho: benchmarks de 2025

Os benchmarks recentes mostram que a diferença de desempenho diminuiu bastante:

```
Métrica                           | ECS             | EKS
----------------------------------|-----------------|----------------
Tempo de inicialização do container | 2-5 segundos    | 3-7 segundos
Máximo de pods por nó             | 250             | 250
Tempo de reação do autoscaling    | 10-30 segundos  | 15-45 segundos
Latência da API do plano de controle | Muito baixa   | Baixa
```

Na maioria dos cenários reais, as diferenças de desempenho são negligenciáveis.

### A tendência de convergência: ECS ficando mais parecido com Kubernetes

Uma coisa interessante em 2025 é como o ECS foi adotando mais recursos parecidos com Kubernetes:

- O ECS agora suporta um subconjunto de construções semelhantes a Pods
- As definições de tarefas ficaram mais parecidas com manifestos Kubernetes
- A AWS adicionou mais capacidades de agendamento inspiradas no Kubernetes

A diferença funcional continua diminuindo. A decisão virou mais sobre ecossistema e modelo operacional do que diferenças de recursos.

### Quando usar ambos: a abordagem híbrida

Algumas organizações estão usando os dois para cargas de trabalho diferentes:

```
ECS para:
- Serviços integrados com AWS
- Microsserviços mais simples
- Cargas de trabalho serverless em containers

EKS para:
- Aplicações stateful complexas
- Necessidades de agendamento especializado
- Equipes com forte preferência por Kubernetes
```

Faz sentido usar cada um onde é mais forte.

### Caminhos de migração: e se você escolher errado?

Uma preocupação comum é ficar preso à escolha errada. A realidade:

**ECS para EKS:**
- Esforço moderado
- Requer mudanças na containerização
- Conversão de CloudFormation/CDK para manifestos Kubernetes

**EKS para ECS:**
- Mais trabalhoso
- Pode exigir mudanças de arquitetura
- Recursos específicos do Kubernetes precisam de alternativas

Projetar as aplicações para minimizar dependências específicas do orquestrador ajuda em qualquer direção.

### Para onde os serviços estão indo

Baseado nos investimentos recentes da AWS e no que vejo na indústria:

**ECS em 2026-2027:**
- Integrações ainda mais profundas com serviços AWS
- Mais recursos inspirados no Kubernetes
- Foco na experiência serverless de containers
- Experiência de desenvolvedor simplificada

**EKS em 2026-2027:**
- Simplificação operacional contínua
- Melhores integrações com serviços AWS
- Capacidades do EKS Anywhere e gerenciamento multi-cloud
- Integração mais profunda com GKE, AKS e DOKS
- Otimização de custos mais sofisticada
- Experiência de desenvolvedor melhorada para reduzir a curva de aprendizado do Kubernetes

A diferença vai continuar diminuindo, mas os trade-offs fundamentais vão ficar.

### Tomando a decisão: uma abordagem prática

Se você está enfrentando essa decisão:

1. **Seja honesto sobre a expertise da sua equipe em Kubernetes**
   Não subestime a curva de aprendizado se sua equipe é nova no Kubernetes.

2. **Avalie seus verdadeiros requisitos multi-cloud**
   Muitas organizações acham que precisam de multi-cloud, mas na prática não precisam.

3. **Liste seus requisitos específicos de orquestração**
   O que você realmente precisa, não o que seria legal ter.

4. **Calcule o custo total de propriedade**
   Inclua custos operacionais, não só as cobranças diretas da AWS.

5. **Considere começar com ECS**
   Para muitas organizações, começar com ECS e migrar para EKS se necessário é menos arriscado que o contrário.

### O veredicto: ainda não existe solução única

Depois de tudo isso, a resposta continua sendo: depende da sua situação.

Em 2025, estes padrões ficaram claros:

- **ECS é a melhor escolha padrão** para equipes focadas em AWS sem requisitos específicos de Kubernetes
- **EKS é a melhor escolha padrão** para equipes com expertise em Kubernetes ou requisitos multi-cloud
- **Os dois serviços estão prontos para produção** e escalam para necessidades empresariais

Nenhuma escolha está errada. São trade-offs diferentes para contextos diferentes.

### Junte-se a mim no AWS Community Day Lisboa 2025

No AWS Community Day Lisboa, vou aprofundar essas escolhas de orquestração de containers com:

- Comparações detalhadas de arquitetura
- Benchmarks de desempenho de implantações reais
- Estratégias de migração entre os serviços
- Técnicas de otimização de custos para as duas plataformas
- Estratégias multi-cloud usando EKS com GKE, AKS e outros serviços Kubernetes
- Frameworks práticos de decisão para seus casos de uso

Se você está começando com containers ou quer otimizar o que já tem, entender esses trade-offs ajuda a tomar decisões melhores.

Vejo você em Lisboa!
