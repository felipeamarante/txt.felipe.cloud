+++
title = "Kubernetes: Debugue Pods Como um Pro"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-03-18"
categories = [
    "K8s",
]
menu = "main"
+++

### Já ficou olhando para um CrashLoopBackOff sem saber o que deu errado?

![meme](/images/evergreen.gif)

Todos nós já passamos por isso—olhando para um pod falhando sem ter ideia do porquê ele não está iniciando. Os logs estão vazios e você está quase perdendo a cabeça.

### Conheça o container de debug efêmero

O Kubernetes 1.23+ tem um recurso que vai mudar sua vida de debugging para sempre:

```bash
kubectl debug -it failing-pod-xyz --image=busybox --target=failing-container
```

Isso anexa um container de debug ao seu pod existente sem reiniciá-lo! Agora você pode inspecionar o sistema de arquivos, verificar variáveis de ambiente e ver o que está realmente acontecendo.

### Mas e se o pod nem sequer iniciar?

Experimente este comando mágico para criar um clone com os mesmos volumes, mas uma imagem diferente:

```bash
kubectl debug failing-pod-xyz -it --copy-to=debug-pod --container=debug-container --image=ubuntu
```

![k8s-debug](/images/k8s-debug.png)

### O kit de ferramentas definitivo para debug

Crie este YAML e mantenha-o sempre à mão:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-toolkit
spec:
  containers:
  - name: debug
    image: nicolaka/netshoot
    command: ['sleep', '999999']
  tolerations:
  - operator: "Exists"
```

Aplique com:

```bash
pbpaste | kubectl apply -f-
```

Agora você tem um pod com todas as ferramentas de rede e debugging imagináveis que pode rodar em qualquer nó.

### Dica pro: Use o plugin kubectl-debug

```bash
kubectl krew install debug
kubectl debug pod failing-pod-xyz
```

Chega de adivinhar o que está acontecendo dentro dos seus pods!