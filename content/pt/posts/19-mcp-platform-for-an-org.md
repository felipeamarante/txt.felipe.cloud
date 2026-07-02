+++
title = "Construindo um Registro de Ferramentas de IA para uma Engenharia Inteira"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-04-21"
categories = [
    "AI",
]
menu = "main"
+++

### "Peraí, a gente tem um servidor MCP pra isso?"

O MCP (Model Context Protocol) é vendido como o "USB-C das ferramentas de IA". Um plug padrão, qualquer modelo, qualquer ferramenta. E essa parte é verdade de verdade. A parte difícil aparece depois.

No trimestre passado fui descobrir quantos servidores MCP existiam na nossa engenharia. A resposta honesta era: ninguém sabia. Algumas equipes tinham servidores próprios. Outras tinham embrulhado servidores de terceiros. Alguns viviam em repositórios privados que duas pessoas tinham favoritado. Nenhuma fonte da verdade sobre o que existia, o que fazia, quais credenciais precisava ou se algum dia tinha passado por revisão de segurança.

Então os agentes fizeram o que agentes fazem quando não têm mapa. Alucinaram ferramentas, ou alguém colou um README meio certo dentro de um prompt e deu por encerrado.

### O problema real não é escrever um servidor

Escrever um servidor MCP é coisa de um fim de semana. Tem template, a spec é pequena, os SDKs são decentes.

Em escala de organização o problema é outro. É governança e descoberta:

- Onde está o servidor?
- Dá pra usar com segurança? Quem decidiu isso?
- Em qual versão eu estou plugando?
- Quais credenciais ele precisa, e como configuro meu cliente?

Nada disso se resolve escrevendo mais servidores. Se resolve tratando o MCP como uma plataforma, não como uma pilha de repositórios. Construí três peças.
### Peça 1: um catálogo só de metadados

O instinto aqui é construir um grande serviço de registro com banco de dados. Resista. Você não precisa de infra em runtime pra cuidar. Você precisa de uma fonte da verdade que seja chata e auditável.

Cada servidor MCP é só uma pasta com um manifesto YAML e um README. O código-fonte continua no repositório dele. O manifesto declara o essencial:

```yaml
name: wiki
description: Read and search the company wiki as MCP tools
version: 1.2.0
remotes:
  - https://example.internal/mcp/wiki
auth:
  required_headers:
    - X-Wiki-Token
```

Um site estático (export estático do Next.js) percorre todos os manifestos no momento do build e renderiza uma página de catálogo privada. Sem servidor, sem banco, sem plantão.

Adicionar um servidor novo é a parte que as pessoas realmente sentem:

1. Copie a pasta `example`.
2. Preencha o manifesto e o README.
3. Abra um PR.

Cerca de 90 segundos depois do merge, ele aparece no catálogo. O versionamento é por servidor, via tags git como `wiki/v1.2.0`, então uma equipe lançando uma versão nova nunca trava outra.

Bônus que se pagou rápido: como o manifesto declara as credenciais necessárias, o catálogo consegue gerar automaticamente a configuração do cliente e as flags de header da CLI pra você. Copia, cola, conectado. Acabou a adivinhação de qual header o servidor queria.

### O catálogo não é só para servidores MCP

Quando o padrão funcionou para servidores MCP, apontei ele para todo o resto que tem o mesmo problema de "onde está isso, e é seguro?". O catálogo acabou guardando três tipos de artefato, todos passando pelo mesmo fluxo de PR e selos.

![Um registro cataloga três tipos de artefato: servidores MCP, plugins e pacotes de contexto de produto, todos pelo mesmo pipeline de PR e selos até um catálogo privado](/images/19-registry-types.pt.svg)

- **Servidores MCP.** Só metadados. O manifesto aponta para um servidor cujo código mora no próprio repo.
- **Plugins.** Plugins do Claude Code que empacotam skills, comandos e agentes. Esses são a exceção: o plugin *é* o artefato, então o conteúdo dele mora no próprio registro e se instala direto do marketplace com um único comando. É como compartilhamos skills reutilizáveis pela organização, em vez de cada um reinventar os mesmos truques de `CLAUDE.md` no seu canto.
- **Pacotes de contexto de produto.** Um pequeno manifesto mapeia um produto para os repos dele, e um job noturno roda o repomix para juntar esse código num arquivo só. Assim, quando alguém escreve um PRD, ou aponta um agente para "o produto de faturamento", dá para fundamentar no código atual de verdade, não numa página de wiki desatualizada.

Mesmo pipeline, mesma governança, três cargas diferentes. O registro deixou de ser uma lista de MCPs e virou o mapa da organização do que a IA realmente pode plugar.

### A parte de governança: segurança mora num arquivo separado

Essa é a parte que eu brigaria pra manter se alguém tentasse cortar.

A decisão de "isso é permitido?" não pode pertencer a quem escreveu o manifesto. Senão todo autor aprova o próprio servidor em silêncio, e seu catálogo vira uma lista de coisas que parecem oficiais mas nunca foram revisadas.

Por isso os selos ficam num arquivo separado: `security_reviewed`, `recommended`, `deprecated`. Esse arquivo é controlado por uma regra de CODEOWNERS, então só o pessoal de segurança pode mudá-lo. O autor abre o PR do servidor dele. Um revisor diferente, e obrigatório, vira o selo.

```
# CODEOWNERS
/catalog/badges.yaml   @security-review
```

Sem aprovações silenciosas. A trilha de auditoria é o próprio histórico do git. Se alguém perguntar "quem disse que esse servidor era seguro e quando", a resposta é um commit, não uma thread de Slack que rolou pra fora da tela seis meses atrás.

### Peça 2: um template pra ninguém começar do zero

Descoberta é metade da batalha. A outra metade é fazer da coisa certa a coisa fácil.

Lançamos um template reutilizável em TypeScript. Você roda e ganha um servidor MCP em minutos: estrutura de projeto consistente, CI já configurado, testes já passando. Mesmo formato sempre, o que significa que qualquer um consegue ler o servidor de qualquer outro sem reaprender um layout novo.

Isso importa mais do que parece. Metade do espalhamento não era má vontade, era atrito. As pessoas faziam servidores improvisados porque montar um "de verdade" parecia um projeto inteiro. Tira esse atrito e os floquinhos de neve esquisitos param de aparecer.

### Peça 3: um servidor de verdade, não um brinquedo

Uma plataforma sem consumidores reais é um slide de apresentação. Então o primeiro servidor que passou pelo pipeline inteiro foi justo o que eu mais queria: a wiki da empresa, exposta aos agentes como ferramentas MCP.

As ferramentas eram as óbvias:

- `read_wiki_page`
- `search_wiki` (busca full-text)
- `list_wiki_index`
- `get_page_relationships`

Algumas decisões que fizeram diferença:

- **Auth**: baseada em token, com hash SHA256 no lado do servidor, um token por cliente pra dar pra revogar um consumidor sem mexer no resto. Cada token carrega uma data de expiração, embora aplicá-la em toda chamada ainda estivesse na minha lista, pra ser honesto.
- **Hospedagem**: serverless em AWS Lambda, atrás de uma function URL do Lambda, com deploy via SAM. Nada pra atualizar entre as requisições.
- **Stateless**: toda chamada é uma consulta ao vivo. A resposta é sempre fresca, porque não tem cache fingindo que a wiki não mudou.

Foi consumido pelo Claude Code, por uma configuração de Copilot Studio e por agentes de CI. E aqui está o retorno: matou uma quantidade enorme de alucinação. Quando um agente passou a consultar a memória real da organização em vez de inventá-la, "acho que o doc diz..." virou "o doc diz, e aqui está a página".

### O que eu te diria se você está prestes a fazer isso

Algumas opiniões, conquistadas do jeito chato:

1. **Metadados primeiro.** Não force todo servidor pra dentro de um monorepo ou de um banco. Uma pasta de manifestos é pesquisável, diffável e de graça.
2. **Builds estáticos.** Sem registro em runtime pra manter vivo. O catálogo é só HTML que reconstrói no merge.
3. **Separe a decisão de segurança da autoria.** CODEOWNERS num arquivo de selos é a governança real mais barata que você vai lançar na vida.
4. **Lance um servidor que as pessoas realmente querem.** A plataforma ganha confiança por um consumidor, não por um README.

O MCP ser o momento USB-C é real. Mas uma porta USB-C que ninguém acha, sem etiqueta dizendo se é seguro plugar, é só um buraco na parede. O encanamento é a parte fácil. O mapa é o trabalho.

Vale fazer direito.
