+++
title = "Eficiência de Contêineres: Por que Um Engenheiro Pode Gerenciar 1000x Mais Contêineres que VMs"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-03-22"
categories = [
    "DevOps",
]
menu = "main"
+++

### Ainda gerenciando VMs como se estivéssemos em 2010?


Todo mundo já viveu isso: provisionar uma VM nova pra cada aplicação, esperar mais de 10 minutos inicializar, depois gastar horas configurando o SO, instalando dependências e resolvendo conflito de pacote. Sem falar em patches e manutenção.

### A proporção engenheiro-recurso que muda os custos

Um engenheiro de operações típico consegue gerenciar em torno de 100-200 máquinas virtuais. Esse mesmo engenheiro pode lidar com mais de 10.000 contêineres. Isso é 50-100x de diferença.

```
# Mundo tradicional de VMs
1 engenheiro : ~100 VMs

# Mundo de contêineres
1 engenheiro : ~10.000 contêineres
```

### Por que a diferença é tão grande?

Contêineres eliminam categorias inteiras de trabalho que consomem tempo no modelo de VMs:

- Gerenciamento de SO: sem patches em 100 instâncias diferentes
- Imutabilidade: sem desvios de configuração, sem servidores "floco de neve"
- Padronização: mesmo processo de deploy pra tudo
- Orquestração: Kubernetes cuida de agendamento, escalonamento e recuperação

### A migração da Expedia

O Grupo Expedia saiu de uma arquitetura monolítica baseada em VMs e foi pra contêineres:

```
ANTES (VMs):
- 50 engenheiros gerenciando ~2.500 VMs
- 2-3 semanas para provisionar novos ambientes
- 80% do tempo gasto em manutenção

DEPOIS (Contêineres):
- 15 engenheiros gerenciando ~8.000 contêineres
- Novos ambientes em minutos
- 70% do tempo gasto em inovação
```

Reduziram custos de infraestrutura em 58% e ainda atenderam 20% mais tráfego.

### Capital One

A migração do Capital One teve resultados parecidos:

```
- 40% de redução nos custos de infraestrutura
- Deploy passou de mensal para diário
- Tempo médio de recuperação (MTTR) caiu de horas para minutos
- Produtividade dos engenheiros aumentou em 300%
```

Faz sentido. Quando a equipe para de apagar incêndio em servidor, sobra tempo pra entregar produto.

### O impacto no custo

Uma empresa de médio porte rodando 500 VMs pode gastar:
```
500 VMs × R$350/mês = R$175.000/mês em infraestrutura
5 engenheiros × R$180.000/ano = R$900.000/ano em pessoal
```

A mesma carga de trabalho em contêineres:
```
Infraestrutura: R$75.000/mês (57% de economia)
Pessoal: 2 engenheiros = R$360.000/ano (60% de economia)
```

Mais de R$720.000 de economia por ano, sem contar o valor de conseguir fazer deploy quando precisa em vez de esperar a janela de manutenção.

### Como começar

Não precisa migrar tudo de uma vez. Escolha um serviço sem estado não crítico pra começar.

1. Construa uma imagem a partir da sua aplicação:
   ```bash
   # Dockerfile simples para uma aplicação Node.js
   FROM node:18-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   CMD ["npm", "start"]
   ```

2. Suba em um serviço gerenciado como AWS ECS, Azure Container Apps ou Google Cloud Run, se ainda não estiver pronto pro Kubernetes.

### Segurança como bônus

Contêineres também melhoram a postura de segurança, não é só eficiência:

- Infraestrutura imutável elimina desvio de configuração
- Superfície de ataque menor com SO mínimo
- Verificações automatizadas no pipeline de CI/CD

### Sem desculpas

No final das contas: menos gasto em infraestrutura, menos gente apagando incêndio em servidor, e deploy saindo quando precisa. Não tem muito o que debater. É hora de começar.
