+++
title = "Meu Segundo Cérebro Esquece de Propósito"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-06-16"
categories = [
    "AI",
]
menu = "main"
+++

### "Peraí, eu realmente decidi isso, ou só anotei uma vez e nunca mais voltei a conferir?"

Eu bato nessa pergunta umas vez por semana. Trabalho em uma dúzia de projetos, tomo uma centena de pequenas decisões e jogo tudo em notas. Seis meses depois abro uma nota que me diz com toda a confiança do mundo algo que deixou de ser verdade lá em março.

Esse é o problema de verdade das notas. Elas não mentem de propósito. Elas só congelam. O fato estava certo quando você escreveu, ninguém atualizou, e agora ele fica ali com a mesma confiança nítida que tinha no primeiro dia. A maioria dos sistemas de notas só sabe acumular. Nunca esquece, e nunca discorda de si mesmo.

Então construí uma base de conhecimento pessoal que faz as duas coisas. Ela esquece de propósito, e mantém as próprias contradições à mostra. É um experimento pessoal, não um produto, mas a ideia se sustentou melhor do que eu esperava.

### O pulo do gato não é "a IA toma notas"

Todo mundo já tentou apontar uma IA para as próprias notas. Funciona por uma semana. Aí o agente cria três páginas levemente diferentes sobre o mesmo projeto, sobrescreve uma decisão da qual você realmente gostava, e começa a alucinar com total confiança porque nada nunca o contestou.

A correção não é um modelo mais esperto. É um contrato mais rígido. Eu dou ao agente um conjunto pequeno de regras que ele não pode quebrar, e a coisa toda para de apodrecer.

### Três camadas, tudo em arquivos simples

Sem banco de dados. Só Markdown, frontmatter YAML e Git para versionamento. Propositalmente sem graça.

1. **`/sources`** é a trilha de auditoria imutável. Os inputs brutos entram aqui e nunca são editados: transcrições de reunião, quadros exportados, áudios despejados, anotações rápidas. Essa é a verdade de base. Se um fato no wiki gerar dúvida, consigo rastrear até o arquivo exato de onde ele veio.

2. **`/wiki`** é a camada editável pelo agente. Uma entidade por página: um projeto, uma pessoa, uma decisão, uma nota diária. Cada página carrega frontmatter que a torna auditável.

3. **`WIKI.md`** é o contrato. É o livro de regras que o agente tem que obedecer toda vez que toca no wiki. Esse arquivo é o produto de verdade. O resto é só armazenamento.

![Base de conhecimento em três camadas: as sources alimentam o wiki, o WIKI.md o governa, e a confiança decai de alta para média e baixa quando uma página fica sem confirmação](/images/23-decay-diagram.pt.svg)

É assim que fica o cabeçalho de uma página do wiki:

```yaml
---
title: "Migração para fora do DMS legado"
type: decision
last_confirmed: 2026-09-12
confidence: high
sources:
  - sources/2026-09-12-arch-review.md
  - sources/2026-08-30-voice-dump.md
---
```

Aquele campo `confidence` é o ponto central, e já chego nele.

### Regra um: componha, não duplique

Quando um fato novo aparece, o agente tem que encontrar-ou-criar a página da entidade correspondente e adicionar um item datado em `## Updates`. Ele nunca tem permissão de criar uma página quase duplicada. Então um projeto não vira "Projeto X", "Projeto X (notas)" e "ProjetoX-final". Vira uma página só que cresce numa linha do tempo.

```markdown
## Updates
- 2026-09-12: Decisão confirmada na revisão de arquitetura. Vamos com a migração faseada.
- 2026-08-30: Inclinado para a migração faseada, ainda nervoso com a camada de dados.
```

Dá para ler uma página de cima a baixo e ver o raciocínio evoluir. Só isso já resolveu metade da minha bagunça de notas.

### Regra dois: mantenha os dois lados de uma contradição

Essa é minha regra favorita. Quando um fato novo conflita com um antigo, o agente não sobrescreve. Ele mantém os dois, com datas.

```markdown
## Updates
- 2026-09-12: A migração agora é faseada ao longo de dois trimestres.
- 2026-06-01: Migração planejada como um único fim de semana big-bang.
```

A maioria dos sistemas substituiria silenciosamente a linha antiga e você nunca saberia que mudou. Eu quero o oposto. Transparência radical. Quando algo contradiz o que eu acreditava três meses atrás, quero ver a costura, não uma mentira lisa. Metade das vezes a própria contradição é a parte interessante.

### Regra três: esqueça no cronograma

Aqui ele deixa de ser um app de notas e começa a parecer memória.

Um job noturno roda e faz três coisas:

- **Arquiva páginas obsoletas.** Qualquer coisa com mais de 45 dias e sem pendências abertas é movida para uma pasta de arquivo. Fora do caminho, ainda no Git, ainda pesquisável, só sem fingir que é atual.
- **Sinaliza wikilinks quebrados.** Se uma página aponta para algo que foi renomeado ou arquivado, isso é sinalizado em vez de ficar pendurado em silêncio.
- **Decai a confiança.** Qualquer página não reconfirmada em 21+ dias cai um nível: `high` vira `medium`, `medium` vira `low`.

Essa última é a estrela. Pensa em como a memória humana funciona de verdade. Você não confia numa lembrança de seis meses do mesmo jeito que confia em algo de ontem. Você hesita. Você diz "acho que foi por volta de X". Minha base de conhecimento faz o mesmo agora. Um fato que ninguém tocou em três semanas fica visivelmente menos confiável, em vez de ficar ali com uma nitidez falsa.

Então quando abro uma página e vejo `confidence: low, last_confirmed: 2026-04-02`, leio de um jeito completamente diferente. Não é "isso é verdade". É "isso era verdade, vai conferir". Os fatos obsoletos envelhecem à vista, em vez de mentir para mim em silêncio.

### A parte que escreveu este post

Tem mais um workflow que eu não esperava amar: a captura de voz. O agente amostra o meu próprio jeito de escrever a partir das sources brutas para um pequeno arquivo de estilo. Então quando ele rascunha algo, sai parecido comigo, não com um assistente genérico que abre todo parágrafo com "No mundo acelerado de hoje".

O que é a confissão honesta aqui: o primeiro rascunho deste post saiu exatamente desse pipeline. Alimentei com o meu arquivo de voz e com minhas próprias notas sobre o sistema, ele me deu algo que soava como eu, e editei a partir dali. Levemente recursivo. Tô de boa com isso.

### Por que arquivos, e por que isso basta

As pessoas perguntam por que não fui de um banco de dados de verdade ou de um daqueles apps chiques de segundo cérebro. Porque a stack sem graça faz tudo de que preciso. Markdown é legível daqui a cinquenta anos. Frontmatter YAML é consultável com cinco linhas de script. O Git me dá uma trilha de auditoria perfeita de graça, cada mudança, cada contradição, cada passo de decaimento, com timestamp e diff. Os agentes de IA só rodam os workflows de auto-organizar e auto-esquecer por cima. Nada fica preso numa caixa proprietária.

É perfeito? Não. O agente de vez em quando arquiva algo na entidade errada, e ainda tenho que dar uma passada de olho no log de contradições. Mas ele já me impediu de confiar em pelo menos três "fatos" que tinham silenciosamente dado errado. É uma boa troca.

A ideia nunca foi construir uma IA que toma notas. Já existem várias. A ideia foi dar à IA um contrato rígido o bastante para que o conhecimento que ela guarda fique mais honesto com o tempo, não menos. O conhecimento deveria ficar menos confiável quando ninguém o reconfirma. Igual ao nosso.

Se as suas notas já mentiram para você sem piscar, tente fazê-las esquecer. De nada.
