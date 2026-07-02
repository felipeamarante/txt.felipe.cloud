+++
title = "Apontei um Agente de IA para uma Base de Código de 20 Anos e Mandei Triar Bugs"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-03-24"
categories = [
    "AI",
]
menu = "main"
+++

### "Ninguém mais entende esse código. Será que a IA entende?"

Foi essa a pergunta que começou tudo. Temos um sistema de gestão de concessionárias que tem mais ou menos vinte anos, escrito em PowerBuilder, com C#/.NET colado por cima ao longo do tempo e um SQL Server por baixo. Quem escreveu a versão original já foi embora, na maioria. A documentação é um boato.

Todo engenheiro sênior que conheço silenciosamente teme os tickets que caem nesse sistema. Então fiz o óbvio e apontei um agente de IA de programação para ele.

Aqui está o que aprendi.

### Código legado é onde os agentes são, estranhamente, mais úteis

O hype diz que a IA é ótima em projetos do zero. Cria um Next.js novinho, monta um CRUD, vê a coisa voar. Tudo bem. Mas esse também é exatamente o lugar onde você, humano, é mais rápido e mais feliz. Você não precisa de muita ajuda ali.

O lugar onde você realmente precisa de ajuda é o lugar que você odeia: 200.000 linhas de PowerBuilder que ninguém lê desde 2009.

E aqui está a sacada. O agente não liga que é PowerBuilder. Ele não suspira ao abrir um arquivo `.srw`. Ele não carrega vinte anos de trauma com essa base de código. Ele lê o código que existe, segue as chamadas entre as linguagens e relata o que encontra. A ausência de documentação, que paralisa um novo contratado humano por semanas, mal o atrasa, porque ele nunca ia ler a documentação mesmo. Não existe documentação.

Essa assimetria é a história toda.

### O agente de triagem de bugs

O primeiro experimento foi um pipeline de triagem. Um ticket cai no Jira, o agente o pega e roda seis passos:

1. Ler o ticket.
2. Decidir se o ticket está completo. (Adianto: metade não está.)
3. Verificar se isso já foi corrigido em uma versão mais recente.
4. Procurar a causa raiz na base de código.
5. Diagnosticar e propor uma correção concreta.
6. Reportar de volta no Jira.

![O pipeline de triagem de bugs em seis passos, com roteamento por confiança](/images/17-triage-pipeline.pt.svg)

Ele conversa com o Jira e com os repositórios de código via MCP (Model Context Protocol), então não está raspando tela de nada. Ele lê tickets e código do mesmo jeito que uma pessoa com o acesso certo leria.

A parte da qual mais me orgulho é o passo 2, que leva ao próximo ponto.

### Roteamento por confiança, ou "pare de inventar correções"

Toda execução de triagem termina com uma nota de confiança de 1 a 5. Na real ela mede o quão longe o agente conseguiu chegar, e decide o que ele está autorizado a fazer em seguida.

```
1 · Sem info suficiente → rebater: pedir passos para reproduzir, uma versão, um erro real.
2 · Afunilou o problema → postar uma análise parcial; sinalizar o que ainda está incerto.
3 · Causa raiz achada   → entregar o diagnóstico a um desenvolvedor.
4 · Correção proposta   → uma correção concreta, ou um PR.
5 · Já corrigido        → apontar a versão que resolveu, e fechar.
```

Essa única decisão de design foi o que tornou a coisa utilizável em vez de perigosa.

Um LLM sem grades de proteção vai "corrigir" com toda a confiança um ticket que diz "às vezes a tela fica esquisita". Ele vai escrever um patch lindo, plausível e completamente fictício. Rotear por confiança inverte esse padrão. Quando o ticket é lixo, o trabalho do agente é dizer isso educadamente e devolver, não alucinar uma correção heroica.

Um bot que rebate um ticket vago está fazendo trabalho de verdade. Um bot que inventa uma correção para um ticket vago está criando incidentes futuros.

### Segunda rodada: um sistema descontinuado sem remote git

Animado, tentei a mesma ideia em um segundo DMS, mais antigo. Esse foi construído em WinDev sobre um backend Oracle, e está oficialmente descontinuado, o que deixou as restrições interessantes:

- Não há remote git upstream. O "repositório" são drops locais de código, marcados com tags no git conforme chegam. Ou seja, não há onde abrir um pull request.
- E, de novo, não havia documentação.

O problema do pull request resolvi fazendo o agente postar a correção como um trecho de código dentro de um comentário no Jira. Menos elegante, mas o revisor recebe a mudança exata para copiar, com o raciocínio anexado. Ele também escreve um log de auditoria em JSONL de cada execução, então quando alguém pergunta "por que o bot disse isso", existe um registro.

Para a documentação faltante, escrevi à mão um pequeno mapa de módulos e um glossário francês-inglês (a base de código é bilíngue da pior forma possível) e injetei isso como contexto. Esse arquivinho fez mais pela precisão do que qualquer ajuste de prompt. O agente não precisa de documentação perfeita. Ele precisa de um mapa do bairro.

### Arqueologia de código

O terceiro experimento abandonou de vez a correção de bugs e só perguntou: um agente consegue documentar um sistema que ninguém entende?

Apontei um para 38 repositórios Java sem documentação. Mais de vinte anos de dívida técnica, nomes de módulos crípticos, dependências entre repositórios, zero documentação. Deixei ele ingerir o código mais uma pilha de arquivos Word e Excel externos que tinham se acumulado ao longo dos anos.

Ele produziu:

- Um documento "passaporte" por módulo. Uma página por módulo: o que faz, com o que conversa, o que o toca.
- Um glossário decodificando os códigos curtos crípticos espalhados por toda parte.
- Uma visão geral do sistema com um grafo de dependências (renderizado em Mermaid, claro).
- E uma lista de coisas sobre as quais não tinha certeza.

Esse último item é a sacada de verdade.

### A honestidade é o produto

Um arquivo de documentação confiante e errado é pior do que nenhuma documentação, porque alguém vai confiar nele e agir. O comportamento mais valioso que tirei desses experimentos foi o agente sinalizar explicitamente a própria incerteza: "Este módulo parece tratar de faturamento, mas encontrei dois caminhos de chamada que não consegui rastrear por completo, e um código curto que não consegui decodificar. Verifique antes de confiar nisto."

Essa frase vale ouro. Transforma o agente de um mentiroso confiante em um júnior útil que conhece os limites do que descobriu. Você consegue construir em cima de um honesto "não tenho certeza". Você não consegue construir em cima de uma mentira fluente.

### Sendo honesto: isso são POCs

Não vou vender peixe demais. Isso são provas de conceito, não mágica. Cometem erros. Precisam de um humano revisando a saída, principalmente nas correções de alta confiança. O glossário e o mapa de módulos exigiram esforço humano real lá no começo. E nada disso substitui entender os seus próprios sistemas.

Mas para o trabalho específico e miserável de triar bugs e mapear código ancião que nenhum humano vivo quer tocar, os agentes pagaram o salário. Eles fazem a arqueologia temida, sinalizam o que não têm certeza e se recusam a inventar correções para tickets sem sentido.

Para PowerBuilder de vinte anos, isso é mais do que eu esperava. Vale tentar.
