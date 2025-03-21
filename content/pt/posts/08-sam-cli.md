+++
title = "SAM CLI: Serverless Simplificado (ou milhões de linhas de terraform?)"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-03-21"
categories = [
    "AWS",
]
menu = "main"
+++

### Ainda implantando funções Lambda manualmente?

![meme](/images/bbq-sim.gif)

Todos nós já passamos por isso—zipando código manualmente, fazendo upload para S3, configurando triggers, configurando API Gateway... é o suficiente para fazer você questionar se serverless realmente vale a pena. Ou pior, escrevendo centenas de linhas de Terraform só para implantar uma função simples.

### Conheça o AWS SAM CLI

AWS Serverless Application Model (SAM) CLI é a ferramenta que vai fazer você se apaixonar novamente pelo desenvolvimento serverless. É como um Docker Compose para funções Lambda.

```bash
# Instalação
brew install aws-sam-cli
```

### A aplicação serverless de 30 segundos

Crie um novo projeto:

```bash
sam init
```

Escolha o "Hello World Example" com Python, e você terá uma estrutura de projeto assim:

```
my-app/
├── hello_world/
│   ├── __init__.py
│   ├── app.py
│   └── requirements.txt
├── events/
│   └── event.json
├── template.yaml
└── tests/
```

A mágica está no `template.yaml`, que define toda sua aplicação:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.9
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

### Teste localmente (sim, é verdade!)

```bash
sam local invoke
```

Ou inicie uma API local (precisa de Docker):

```bash
sam local start-api
```

### Deploy com um único comando

```bash
sam deploy --guided
```

Isso compila seu código, empacota, cria/atualiza a stack do CloudFormation e te dá um endpoint de API. Tudo com um comando!

### Dica pro: Use SAM Accelerate para iterações mais rápidas

```bash
sam sync --watch
```

Isso observa as alterações nos arquivos e automaticamente sincroniza com a nuvem—sem necessidade de reimplantação completa.

Sem mais desculpas para não usar serverless. O SAM CLI torna tudo quase fácil demais!