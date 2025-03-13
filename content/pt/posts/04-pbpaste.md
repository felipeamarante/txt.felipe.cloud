+++
title = "Como dar ctrl+v direto para seu cluster k8s no mac"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-03-13"
categories = [
    "K8s",
]
menu = "main"
+++

### Então você está seguindo um tutorial longo...

E você está ficando completamente cansado de copiar e colar todos os YAMLs... urgh.  
E se eu te dissesse que **posso tornar seu trabalho 50% menos doloroso?** Você me pagaria um café? ☕

Dê uma olhada no YAML abaixo  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: definitely-not-prod
  labels:
    owner: "friday-5pm-dev"
spec:
  containers:
    - name: chaos-container
      image: busybox
      command: ["sh", "-c", "echo 'Implantando... brincadeira!' && sleep 3600"]
      lifecycle:
        postStart:
          exec:
            command: ["/bin/sh", "-c", "echo 'Contexto configurado corretamente?'"]
        preStop:
          exec:
            command: ["/bin/sh", "-c", "echo 'Adeus, cluster cruel!'"]
  restartPolicy: Never
  nodeSelector:
    coffee-supply: "critical"
```
Você pode simplesmente copiar o YAML acima e usar `pbpaste` para enviar seu clipboard diretamente para o `kubectl apply` com o comando abaixo:

```bash
pbpaste | k apply -f-
```

Tchã-rã! Funciona!

![pbpaste](/images/pbpaste.png)

Agora tenha cuidado com o que você copia 🫡