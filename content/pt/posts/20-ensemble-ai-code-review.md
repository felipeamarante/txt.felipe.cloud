+++
title = "Code Review em Ensemble: Vários Revisores de IA, Um Só Juiz"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-05-05"
categories = [
    "DevTools",
]
menu = "main"
+++

### "É só deixar a IA revisar o PR" parece ótimo, até você ler os comentários

Code review é o gargalo que ninguém quer admitir. É lento, é inconsistente, e a qualidade depende inteiramente de quem estava livre naquela tarde. Então o movimento óbvio em 2026 é jogar um LLM no problema.

Eu tentei. Foi um desastre.

Aponte um modelo para um diff e peça que ele revise, e ele vai alegremente despejar vinte comentários. Metade são picuinhas de formatação que o seu linter já pegou. Alguns estão confiantemente errados. Um ou dois são realmente úteis, enterrados em algum lugar no meio. Os desenvolvedores aprendem em uma semana que o bot está dando alarme falso, e aí param de ler de vez. Pior que inútil: é ruído com a confiança de um robô.

Então reconstruí tudo em volta de uma ideia diferente.

### O problema não são os revisores. É que ninguém os filtra.

Tem uma coisa sobre revisores de IA: mais deles na verdade dá uma cobertura melhor. Um modelo perde a injeção de SQL mas pega a race condition. Outro aponta a verificação de null que faltou. Uma persona focada em segurança vê coisas que uma genérica passa batido. Diversidade ajuda.

O detalhe é que diversidade também multiplica o ruído. Cinco revisores significam cinco vezes mais picuinhas, cinco vezes mais falsos positivos, cinco chances de estar confiantemente errado. Se você manda tudo isso direto para o PR, você piorou o problema do spam, não resolveu.

Num time humano você nunca faria isso. Você não posta as anotações cruas de cada júnior para o autor. Alguém sênior lê o feedback, joga fora o que está errado, e diz a única coisa que importa.

É esse o padrão. Muitos olhos, um juiz.

### O design: espalhar, depois julgar

O sistema tem duas fases distintas, e a segunda é onde mora todo o valor.

**Fase um: espalhar.** Um roster (só um arquivo YAML) define quais revisores rodam em um dado diff. Cada um é uma persona de IA separada com seu próprio foco: segurança, performance, corretude, estilo, o que você quiser. Eles rodam como jobs assíncronos em paralelo, e cada revisor explora o repositório de verdade de forma independente. Não só as linhas do diff, mas o código em volta, os imports, os pontos de chamada. Cada um emite seus achados como dados estruturados, não como prosa. Um achado tem arquivo, linha, severidade, um título curto, a justificativa e uma correção concreta.

**Fase dois: julgar.** Essa é a parte que faz funcionar.

Primeiro, os achados crus são mesclados por voto. Se três revisores apontam a mesma linha de forma independente, isso é um sinal forte. Se um revisor menciona algo que ninguém mais notou, é mais fraco, mas fica guardado como candidato.

Depois um único modelo forte, o Review Lead, assume. Eu uso o Claude Opus para esse, porque o juiz precisa ser mais esperto que os revisores. O Lead recebe os achados candidatos mesclados E o diff real, e seu único trabalho é julgar:

- Descartar os falsos positivos. O revisor alegou um null pointer, mas a variável está protegida três linhas acima. Fora.
- Descartar o irrelevante. Está correto, mas é sobre código que o diff não tocou. Fora.
- Ranquear o que sobrou por quantos revisores o apontaram, depois severidade, depois o quão concreta é a correção.
- Escrever exatamente UM comentário.

Uma revisão. Um comentário no GitHub. O sinal, nada da lama.

![Fluxo de review em ensemble: o diff se espalha para revisores independentes, os votos se juntam, um juiz descarta falsos positivos e emite um único comentário consultivo](/images/20-ensemble-review.pt.svg)

### Apenas consultivo, de propósito

O bot nunca bloqueia um merge. Nunca.

Isso é uma escolha deliberada de confiança, não uma limitação técnica. No momento em que um revisor de IA pode bloquear seu PR, duas coisas ruins acontecem. As pessoas começam a burlar ele, e as pessoas começam a odiar ele. Um revisor consultivo que acerta 90% das vezes é um presente. Um revisor bloqueante que erra 10% das vezes é um imposto diário que o time inteiro vai contornar dentro de um mês.

Então ele aconselha. Os humanos ainda decidem. O bot conquista confiança sendo útil, não segurando o botão de merge como refém.

### As partes chatas que tornam isso real

Um brinquedo roda num notebook. Produção tem restrições, e as restrições são a maior parte do trabalho.

**Residência de dados.** A orquestração é Node.js no AWS Lambda, deployada com SAM. Cada bit de inferência roda no Bedrock dentro da região, no nosso caso eu-west-1. O código e os diffs nunca saem da região. Para uma empresa europeia revisando código proprietário, isso não é um luxo, é a diferença entre "pode" e "o jurídico disse não."

**Menor privilégio.** Dois tokens separados, nunca um. Um token de organização somente-leitura que pode clonar e ler repositórios, e nada mais. Um token separado, com escopo de revisão, que pode postar o único comentário, e nada mais. Se um vazar, o raio de dano é minúsculo.

**Segurança do webhook.** Todo webhook que chega tem sua assinatura HMAC validada antes de qualquer coisa rodar. O acesso é restrito a membros, então um fork-and-PR aleatório da internet não dispara uma revisão às suas custas.

**Trilha de auditoria.** Toda execução loga suas métricas (quais revisores rodaram, o que acharam, o que o Lead manteve ou matou) em uma branch dedicada. Esse log é ouro. É como você descobre quais revisores estão puxando o peso e quais só geram ruído, e é como você ajusta o roster ao longo do tempo em vez de adivinhar.

### O que eu te diria antes de você construir o seu

O instinto é tornar os revisores mais espertos. Resista. Os revisores podem ser meramente bons. A alavanca está no juiz.

Um painel mediano com um árbitro afiado ganha de um painel de gênios sem filtro, toda vez. O árbitro é o que transforma "vinte comentários, metade errados" em "um comentário, que vale a pena ler." Gaste o seu melhor modelo, o seu melhor prompt e a sua avaliação mais cuidadosa ali.

Muitos olhos acham os bugs. Um juiz faz com que valha a pena escutar.

Vale construir.
