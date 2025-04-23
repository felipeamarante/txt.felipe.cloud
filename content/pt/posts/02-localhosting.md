+++
title = "Como compartilhar seu localhost"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-01-09"
categories = [
    "K8s",
]
menu = "main"
+++

### Ei, dê uma olhada no meu site...

![meme](/images/localhost01.png)

Recentemente, eu estava implantando algo no meu cluster Kubernetes Kind e me perguntei como poderia mostrar o trabalho para meu colega. No passado, brinquei com o [ngrok](https://ngrok.com) para expor meu localhost para a internet, mas no Kubernetes eu precisaria fazer algum portforwarding complicado, o que não era uma opção.

Então vi que o [Ngrok lançou um operador](https://github.com/ngrok/ngrok-operator) que pode expor facilmente seus containers para a web com uma simples criação de um Ingress.

![ngrok](/images/localhost02.png)

É bem simples de usar, passos são,
 1. Crie uma [conta gratuita no Ngrok](https://dashboard.ngrok.com/signup)
 1. [Instale o operador](https://ngrok.com/docs/getting-started/kubernetes/?k8s-install=ingress#step-1-install-kubernetes-operator)
 1. Implemente o que quiser expor - Exemplo, [Podinfo](https://github.com/stefanprodan/podinfo)
 1. Crie um ingress como o exemplo abaixo, apenas substitua o host pelo seu Ngrok **Nome de Domínio Completo**

 ```yaml
 apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  ingressClassName: ngrok
  rules:
    - host: SEU_DOMINIO_NGROK_AQUI
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: example-service
                port:
                  number: 80
```

Espere alguns segundos e você deverá obter a URL ao descrever o ingress que acabou de criar

```bash
kubectl get ing -A
```

Feliz Localhosting.