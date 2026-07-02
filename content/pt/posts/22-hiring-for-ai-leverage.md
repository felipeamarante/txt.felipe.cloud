+++
title = "Contratar por Alavancagem de IA, Não por Linhas de Código"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-06-02"
categories = [
    "AI",
]
menu = "main"
+++

### "Consegue inverter uma lista encadeada no quadro branco?"

Em 2026, essa pergunta está morta. Ajudei a desenhar um processo seletivo para uma vaga de engenharia sênior+ recentemente, e a primeira coisa que jogamos fora foi o quebra-cabeça de código.

Aqui vai a verdade incômoda que ninguém quer dizer em voz alta: a IA escreveu 80% do código de produção que entreguei no último trimestre. Não o boilerplate. Funcionalidades de verdade. Então por que eu ainda estaria avaliando candidatos por escreverem uma busca binária mais rápido que a pessoa anterior? Esse sinal deixou de corresponder ao desempenho no trabalho faz um tempo.

A pergunta que importa agora é outra. Não é "você consegue escrever a funcionalidade?", mas "você consegue construir o sistema que deixa a IA continuar escrevendo as funcionalidades depois que você saiu de cena?"

### O que estávamos realmente avaliando

A vaga era de um IC sênior, o tipo de pessoa que define a direção técnica de uma linha de produto inteira. Nesse nível, o gargalo não é velocidade de digitação. Nunca foi. É julgamento, alavancagem e a capacidade de multiplicar todo mundo ao redor (inclusive as máquinas).

Então paramos de pedir para o candidato ser a pessoa mais inteligente da sala. Pedimos para ele construir a coisa que deixa uma sala mais burra inteligente.

Chamamos isso de alavancagem de IA. Você consegue montar o scaffolding para que um agente de IA, daqui a seis meses, estenda esse produto sem ninguém segurando a mão dele?

### O formato: uma fatia pequena, em um dia

Eis o desafio que definimos. Construa uma fatia pequena de um app de domínio realista. O candidato escolhe o módulo. Estoque, agendamento, faturamento, o que ele quiser, desde que seja um pedaço crível de um produto real.

Mas o app é só o portão de entrada. É passa ou não passa, e a barra é baixa. Roda? Não é lixo? Ótimo, segue o jogo.

Duas exigências firmes, porque são baratas de declarar e filtram os enroladores em silêncio: a fatia tem que estar **deployada num free tier** (Vercel, Supabase, um EC2 de free tier, o que for) e acessível por uma URL, e precisa ter **auth básica**. Se você não consegue subir e proteger a sua própria fatia pequena, a conversa sobre scaffolding perde o sentido.

O app não é o que avaliamos. Avaliamos o scaffolding de engenharia de IA ao redor dele:

1. **Regras e prompts de IA.** Um arquivo de instruções de agente de verdade (um bom `CLAUDE.md`, ou o que sua ferramenta usar). Ele codifica as convenções, os perrengues, a arquitetura, para que o agente não reinvente a roda a cada prompt?
2. **Skills e subagentes reutilizáveis.** A pessoa extraiu o trabalho repetitivo para coisas que um agente pode chamar? Ou é um prompt gigante e uma reza?
3. **Agentes de dia 2.** Geração de funcionalidades, triagem de bugs, operação. O que uma equipe de fato convive depois que o MVP sobe.
4. **Loops de verificação e avaliação.** Como o sistema sabe que a IA fez a coisa certa? Testes, evals, guardrails. Essa é a parte que separa quem realmente rodou IA em produção de quem assistiu a uma demo.

O MVP leva umas duas horas. O scaffolding é onde um engenheiro sênior gasta o dia, e é onde mora o sinal de verdade.

### A parte sobre a qual todo mundo está em pânico silencioso

Os desafios para casa estão quebrados. Todo mundo sabe. Você passa um problema para alguém, a pessoa passa para uma IA, a IA devolve uma solução polida e você não aprende nada sobre o ser humano. A indústria inteira finge que isso não está acontecendo.

Então adicionamos uma virada ao vivo no dia 2.

Na apresentação, o candidato nos mostra o que construiu. Aí entregamos uma tarefa não vista. Adicione um campo de status. Corrija este bug. Estenda a UI para fazer X. E assistimos ele fazer, ao vivo, na nossa frente, usando o próprio scaffolding.

Isso resolve duas coisas ao mesmo tempo, e as duas são enormes.

Primeira, mata o problema da cola de vez. Não importa quem ou o que escreveu o desafio para casa. Se o scaffolding for genuinamente bom, o candidato voa na tarefa ao vivo porque o próprio sistema o carrega. Se ele fingiu, congela. Dá para ver em tempo real. Acabou a adivinhação de se aquele PDF polido era mesmo dele.

Segunda, prova que o sistema entrega, não só faz demo. Boa parte da tooling de IA parece incrível no caminho feliz e desmorona no instante em que a realidade foge do roteiro. A tarefa de dia 2 é a realidade fugindo do roteiro. Se o `CLAUDE.md`, os subagentes e o loop de avaliação realmente tornam a mudança não vista fácil, isso é o trabalho inteiro, demonstrado ao vivo.

Foram os melhores quarenta e cinco minutos de sinal que tirei de uma entrevista em anos.

### O rubric (e a gente entrega de antemão)

Sem jogos mentais. Os candidatos recebem o rubric antes de começar. Eis a ponderação que usamos:

```
Alavancagem de IA & qualidade dos artefatos   45%
Extensibilidade no dia 2 (ao vivo)            25%
Julgamento de engenharia                      20%
Comunicação                                   10%
```

A maior fatia é o scaffolding. A segunda maior é se ele aguentou ao vivo. Juntas, são 70% da nota apostadas em "você construiu alavancas que funcionam", que é exatamente onde queríamos.

Compartilhar o rubric de antemão importa por mais do que justiça. Diz algo sobre a pessoa. Um candidato que lê "45% alavancagem de IA" e mesmo assim passa o dia inteiro polindo o app em vez do scaffolding já respondeu a uma pergunta que você nem precisou fazer.

E todo mundo recebe uma tarefa de dia 2 comparável, mas fresca. Mesma dificuldade, especificidades diferentes, para que ninguém passe a resposta para o próximo candidato no corredor. Justo e resistente a viés, por design. Do nosso lado existe um guia de entrevistador correspondente: âncoras de nota de 1 a 5 para cada linha do rubric, para que dois entrevistadores cheguem mais ou menos ao mesmo lugar, e um pequeno banco de tarefas de dia 2 calibradas para a mesma dificuldade.

### A lição, se você desenha entrevistas

Pare de avaliar o artefato que um engenheiro sênior produz à mão. Em 2026, isso é a coisa mais barata que ele faz.

Avalie a alavancagem. Ele consegue construir o sistema que continua produzindo depois que sai da sala? Consegue fazer uma mudança fresca e não vista acontecer na sua frente usando só o scaffolding que ele mesmo construiu? Esse é o trabalho agora. É o trabalho inteiro.

A busca binária no quadro branco te dizia que alguém estudou na véspera. A tarefa de dia 2 ao vivo te diz se a pessoa consegue de fato fazer a IA levar um produto adiante. Só um dos dois vale o seu tempo.

Redesenhe seu processo de acordo. De nada.
