+++
title = "Kubernetes: Debugue Pods Como um Pro"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-02-18"
categories = [
    "K8s",
]
menu = "main"
+++

### Já ficou olhando para um CrashLoopBackOff sem saber o que deu errado?

![meme](/images/evergreen.gif)

Todos nós já passamos por isso: olhando para um pod falhando sem ter ideia do porquê ele não está iniciando. Os logs estão vazios e você está quase perdendo a cabeça.

### Container de debug efêmero

O Kubernetes 1.23+ tem um recurso bem útil pra isso:

```bash
kubectl debug -it failing-pod-xyz --image=busybox --target=failing-container
```

Isso anexa um container de debug ao seu pod existente sem reiniciá-lo. Você consegue inspecionar o sistema de arquivos, checar variáveis de ambiente e rodar comandos dentro do contexto do container.

### Mas e se o pod nem sequer iniciar?

Esse comando cria um clone com os mesmos volumes, mas com uma imagem diferente:

```bash
kubectl debug failing-pod-xyz -it --copy-to=debug-pod --container=debug-container --image=ubuntu
```

### Kit de ferramentas de debug

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

Com isso você tem um pod com ferramentas de rede e debugging que roda em qualquer nó.

### Dica pro: Use o plugin kubectl-debug

```bash
kubectl krew install debug
kubectl debug pod failing-pod-xyz
```

Com isso você já tem o que precisa pra parar de adivinhar.
