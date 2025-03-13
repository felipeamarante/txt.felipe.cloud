+++
title = "Um jeito facil de lidar com multi-account no awscli"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-03-12"
categories = [
    "AWS",
]
menu = "main"
+++




### Você já executou um comando na conta errada?

![meme](/images/this-is-fine.gif)

Isso já me aconteceu uma vez—implantei uma stack de produção na conta de sandbox e não fazia ideia do porquê não estava funcionando... Transformando essa experiência frustrante em algo útil, decidi criar uma ferramenta que me ajuda (e potencialmente ajuda você) a definir um perfil ou contexto, semelhante ao [kubectx](https://github.com/ahmetb/kubectx).

É uma função simples de shell que você adiciona ao seu `.bashrc` ou `.zshrc`, e ela carrega sempre que você inicia uma nova sessão de shell. O nome dela é `awsctx` (sim, muito criativo).

![awsctx](/images/awsctx.png)

### Como instalar?

Por favor, consulte meu README.md super bem documentado neste repositório! E não se esqueça de deixar uma estrela.

https://github.com/felipeamarante/awsctx
