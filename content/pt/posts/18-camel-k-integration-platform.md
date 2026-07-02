+++
title = "Arrastando Integrações Legadas Para o Kubernetes Com Apache Camel K"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-04-07"
categories = [
    "AWS",
]
menu = "main"
+++

### "Por que uma única integração vive em cinco linguagens diferentes?"

Porque ninguém planejou. Simplesmente cresceu assim.

Trabalho em uma empresa de software automotivo, e um canto do nosso mundo conecta fabricantes de carros (os OEMs) a sistemas de gestão de concessionárias. Cada fabricante tem seus próprios padrões de API, seu próprio esquema de autenticação, sua própria ideia do que é um "pedido de veículo". Multiplique isso por décadas de "manda logo" e você tem integrações espalhadas por C#/.NET, Java, VB.NET e, sim, algumas coisas ainda rodando em ASP Classic.

Funciona. Essa é a parte perigosa. Funciona bem o suficiente para ninguém querer mexer, e cada novo parceiro OEM adiciona mais um fio ponto a ponto ao espaguete.

Esse é o assunto pouco glamouroso. Também é onde uma quantidade assustadora de valor empresarial fica presa.

### O formato da bagunça

Imagine três problemas empilhados um sobre o outro:

- **A borda é o caos.** Cada parceiro bate em um endpoint diferente, com um contrato diferente e uma forma diferente de autenticar. Não existe uma porta de entrada única.
- **A lógica está enterrada.** As regras de integração de verdade (transforma este campo, enriquece com aquela consulta, faz retry neste erro, cai para aquela fila) vivem dentro de código compilado em cinco linguagens. Você não consegue ver. Não consegue fazer diff. Mal consegue alocar gente.
- **Os backends legados são radioativos.** Procedures SQL, interfaces DLL, coisas que você cutuca com muito cuidado e nunca olha de frente.

Qualquer modernização que ignore até uma dessas camadas fracassa. Não dá para colocar um gateway bonito na frente de uma bagunça e chamar de moderno.

### A plataforma: gateway na frente, Camel no meio, adapters isolando o legado

A arquitetura-alvo divide o problema de forma limpa nessas mesmas três camadas. Ainda não chegamos lá por completo (mais sobre isso já já), mas é essa a forma para a qual estamos migrando.

**A borda: um API gateway.** Um único ponto de entrada HTTPS padronizado (usamos Gravitee), na frente de mais de 130 endpoints, em dezenas de domínios, com autenticação multi-tenant. Os parceiros param de se importar com onde o backend mora. Eles ganham uma porta de entrada, um modelo de autenticação, um lugar para aplicar rate limits e políticas. Só isso já eliminou uma categoria inteira de chamados do tipo "qual endpoint eu chamo mesmo?".

**O meio: Apache Camel K.** Essa é a parte de que eu realmente quero falar. O Camel K roda rotas do Apache Camel nativamente no Kubernetes, definidas de forma declarativa em YAML. Todos aqueles Enterprise Integration Patterns que você costumava escrever na mão (multicast, agregação, roteamento baseado em conteúdo, tratamento de erros, fallbacks) viram configuração legível em vez de código enterrado. A lógica de integração deixa de ser um mistério dentro de uma DLL e vira um arquivo no git.

**A base: uma camada de adapters de DMS.** Essa é a quarentena. Os adapters falam SQL e DLL com os backends legados feios de um lado, e contratos limpos com a plataforma cloud do outro. A parte radioativa fica atrás do vidro.

O gateway padroniza a borda. O Camel padroniza o meio. Os adapters isolam o legado. Três camadas, três funções.

![Plataforma de integração em camadas: parceiros OEM, API gateway, rotas Camel K no Kubernetes, camada de adapters DMS, backends legados](/images/18-camel-k-arch.pt.svg)

### Como uma rota Camel K se parece de verdade

Aqui vai um exemplo enxuto: um endpoint que se espalha para vários backends em paralelo e agrega as respostas. É o tipo de EIP (multicast mais agregação) que antes era umas poucas centenas de linhas de código com threads.

```yaml
apiVersion: camel.apache.org/v1
kind: Integration
metadata:
  name: vehicle-lookup
spec:
  flows:
    - from:
        uri: "platform-http:/vehicle/{vin}"
        steps:
          - multicast:
              aggregationStrategy: "#mergeStrategy"
              parallelProcessing: true
              steps:
                - to: "direct:oem-inventory"
                - to: "direct:dealer-pricing"
                - to: "direct:warranty-status"
          - marshal:
              json: {}
          - setHeader:
              name: "Content-Type"
              constant: "application/json"
    - from:
        uri: "direct:oem-inventory"
        steps:
          - to: "https://backend.internal/inventory?bridgeEndpoint=true"
          - onException:
              handled: { constant: true }
              steps:
                - setBody:
                    constant: '{"inventory": null, "degraded": true}'
```

Você consegue ler isso. Um engenheiro novo consegue ler isso. Você revisa num pull request, faz grep, vê exatamente o que acontece quando o `oem-inventory` cai (ele degrada com elegância em vez de derrubar a resposta inteira). Compare com encontrar a mesma lógica espalhada entre um serviço em C# e um helper em VB.NET.

### Strangler fig, não big bang

Aqui vai a regra pela qual eu morro: nada de cutover big-bang. Nunca.

Integrações antigas e novas coexistem na mesma plataforma. Hoje a maioria das integrações ainda roda no runtime antigo (uma mistura de Azure Functions e lógica embutida no próprio DMS); o Camel K é onde caem as rotas novas e as recém-migradas. O gateway roteia parte do tráfego para uma rota Camel K novinha e o resto para o caminho legado, e movemos os endpoints um de cada vez. É o padrão strangler fig (figueira-estranguladora), e é a única forma sã de modernizar algo que está ativamente dando dinheiro. Se a rota 37 der problema, você volta aquele endpoint específico. Ninguém recebe ligação às 3 da manhã porque "a migração" falhou como um evento único. Não existe evento único.

É mais lento. Também é a diferença entre uma modernização que entra em produção e uma que vira slide de lição aprendida.

### As partes de IA (na medida certa)

Alguns toques que mereceram o lugar. Vou ser breve, porque a história aqui é a arquitetura de integração, não a IA.

- **Um pipeline de migração com IA.** Esse é o que realmente puxa peso. Uma cadeia de agentes escaneia uma integração legada, tria e avalia o que ela realmente faz, rascunha a rota Camel K equivalente e roda uma checagem de conformidade contra os nossos padrões, com dashboards acompanhando o quanto da migração já andou. Ele não migra nada sozinho. Transforma "faça engenharia reversa nessa integração VB.NET sem documentação na mão" em "revise o que o agente rascunhou", que é uma terça-feira bem diferente.
- **Uma wiki que se mantém atualizada.** Um LLM mantém a documentação das integrações em sincronia conforme as rotas evoluem. O fracasso clássico de plataformas de integração é a documentação apodrecer no instante em que a primeira rota muda. Amarrar a geração de docs às definições das rotas faz a wiki realmente refletir a realidade.
- **Um servidor MCP sobre os endpoints do DMS.** Expomos as operações do DMS como ferramentas do Model Context Protocol, para que agentes possam ajudar a escrever e operar integrações contra endpoints reais (em sandbox). Combinado com agentes do Claude Code que montam o esqueleto das rotas Camel, geram testes e escrevem o primeiro rascunho da documentação, os 70% chatos de uma nova integração ficam bem menos chatos.

É isso. Nada de "a IA transforma tudo". Ela só elimina o trabalho braçal.

### Como eu de fato começaria (pode roubar)

Você não precisa de um cluster nem de orçamento para aprender isso. Eu prototipei o formato inteiro localmente primeiro.

```bash
# Kubernetes local dentro do Docker
kind create cluster --name camel-poc

# Instala o operador do Camel K via Helm
helm repo add camel-k https://apache.github.io/camel-k/charts/
helm install camel-k camel-k/camel-k --namespace camel-k --create-namespace

# Faz deploy de uma rota
kamel run vehicle-lookup.yaml
```

Monte aquele exemplo de espalhar-e-agregar contra algumas APIs públicas. Vai dar deploy. Vai dar uma sensação ótima.

Depois faça a parte que todo mundo pula: **day-2 ops.** Porque "deu deploy" é onde o trabalho de verdade começa, não onde termina.

- **Health probes.** Configure readiness e liveness para o Kubernetes realmente saber quando uma rota travou.
- **Escala.** Observe como as rotas se comportam sob carga. Decida o que escala horizontalmente e o que não escala.
- **Métricas.** Coloque o Prometheus coletando as métricas das rotas. Você quer ver throughput e taxa de erro por integração, não adivinhar.
- **Patching e upgrades.** Pratique atualizar o operador. Pratique fazer rollout de uma rota sem perder mensagens.
- **Troubleshooting.** Quebre algo de propósito. Leia os logs. Aprenda onde o Camel esconde os erros antes da produção te ensinar.

### A parte honesta

Camel K não é almoço grátis. Tem uma curva de aprendizado real: a DSL do Camel, o vocabulário de EIP, as manias do operador. As primeiras rotas parecem mais lentas do que só escrever o código que você já sabe escrever.

E day-2 ops é o trabalho de verdade. Qualquer um consegue fazer uma rota dar deploy. Rodar mais de 100 delas de forma confiável, com observabilidade, degradação graciosa e uma estratégia de patching, é o que você está de fato assinando embaixo.

Mas o retorno é real: a lógica de integração vira YAML declarativo, versionado e observável em vez de conhecimento tribal enterrado em cinco linguagens. A borda fica padronizada. O legado fica em quarentena. E você migra um endpoint de cada vez, então nunca aposta o negócio inteiro numa única noite assustadora.

Pouco glamouroso? Totalmente. Vale a pena? Também totalmente.
