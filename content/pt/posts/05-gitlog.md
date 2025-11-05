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

Depois de anos franzindo os olhos para hashes de commits e carimbos de data/hora, finalmente encontrei a solução que mudou tudo.

### O único alias do git que você realmente precisa

```bash
git config --global alias.lg 'log --graph --pretty=format:"%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset" --abbrev-commit'
```

Agora é só digitar `git lg` e contemplar:

![git-log](/images/git-log.png)

### O que torna isso tão bom?

- **Ramificação visual** com a opção `--graph`
- **Informações coloridas** que são realmente legíveis
- **Formato compacto** mostrando apenas o que você precisa
- **Referências de branches** claramente visíveis
- **Tempo relativo** para você saber se foi "há 2 dias" em vez de algum timestamp

### Como configurar?

Basta copiar e colar esse comando no seu terminal. É só isso.

Se preferir, adicione diretamente ao seu arquivo `.gitconfig`:

```
[alias]
    lg = log --graph --pretty=format:"%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset" --abbrev-commit
```

Agora você nunca mais vai se perder no histórico do git. De nada.