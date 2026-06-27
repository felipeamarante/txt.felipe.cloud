+++
title = "O alias do git que mudou minha vida"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-02-02"
categories = [
    "DevTools",
]
menu = "main"
+++

### Já se perdeu no log do git?

![meme](/images/lost-in-code.png)

Todos nós já passamos por isso: tentando descobrir o que aconteceu em um projeto olhando fixamente para a saída padrão do `git log`. É como tentar ler um romance escrito em papel de recibo.

Depois de anos franzindo os olhos para hashes de commits e timestamps, um alias resolveu tudo isso.

### O único alias do git que você realmente precisa

```bash
git config --global alias.lg 'log --graph --pretty=format:"%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset" --abbrev-commit'
```

Agora é só digitar `git lg` e contemplar:

![git-log](/images/git-log.png)

### Por que funciona?

O `--graph` desenha a árvore de branches no terminal. As cores separam hash, mensagem, autor e tempo sem precisar decifrar nada. E o tempo relativo ("há 2 dias") é muito mais útil do que um timestamp ISO no meio de uma tela monocromática.

### Como configurar?

Cola o comando no terminal. Se preferir, adicione direto no `.gitconfig`:

```
[alias]
    lg = log --graph --pretty=format:"%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset" --abbrev-commit
```

Agora você nunca mais vai se perder no histórico do git. De nada.
