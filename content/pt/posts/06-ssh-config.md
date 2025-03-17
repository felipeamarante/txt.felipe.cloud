+++
title = "SSH Config: O arquivo que você tem ignorado"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-03-13"
categories = [
    "DevTools",
]
menu = "main"
+++

### Você ainda digita comandos SSH completos como se estivéssemos em 2005?

![meme](/images/typing-furiously.gif)

Todos nós já passamos por isso—digitando os mesmos comandos SSH repetidamente:

```bash
ssh -i ~/.ssh/special_key.pem user@ec2-54-234-56-78.compute-1.amazonaws.com
```

### Conheça o mágico arquivo ~/.ssh/config

```
# ~/.ssh/config
Host bastion
    HostName ec2-54-234-56-78.compute-1.amazonaws.com
    User ec2-user
    IdentityFile ~/.ssh/special_key.pem

Host prod-db
    HostName 10.0.5.12
    User admin
    ProxyJump bastion
```

Agora você pode simplesmente digitar:

```bash
ssh bastion
# ou
ssh prod-db
```

### Por que isso muda tudo:

- **Aliases legíveis** em vez de endereços IP
- **Seleção automática de chaves** sem a flag -i
- **Hosts de salto** configurados uma vez, usados para sempre

Seu futuro eu vai agradecer por ter dedicado 5 minutos para configurar isso hoje!