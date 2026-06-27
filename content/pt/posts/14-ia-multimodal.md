+++
title = "IA Multimodal: Quando LLMs Apenas de Texto Não São Suficientes"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-04-15"
categories = [
    "IA",
]
menu = "main"
+++

### "Você pode me dizer o que há de errado com este diagrama de circuito?"

![diagrama de circuito](/images/circuit.gif)

Enviei esta mensagem ao ChatGPT junto com uma foto de uma placa de circuito que eu estava tentando consertar. A resposta? "Desculpe, mas não consigo ver nenhuma imagem que você compartilhou."

Certo. IA apenas de texto tem seus limites.

A última geração de modelos, os chamados multimodais, consegue ver imagens, ouvir sons e até entender vídeos. E dá pra usar já.

### O que é IA multimodal?

LLMs tradicionais, como as primeiras versões do GPT, entendem apenas texto. Modelos multimodais podem processar e gerar múltiplos tipos de dados:

```
LLM apenas de texto:
Entrada: Texto → Modelo → Saída: Texto

IA Multimodal:
Entrada: Texto + Imagens + Áudio → Modelo → Saída: Texto + Imagens + Áudio
```

### Quatro capacidades que valem atenção

Depois de experimentar bastante com GPT-4V, Claude 3 e Gemini, estas são as que mais usei na prática:

1. Compreensão visual: analisar gráficos, diagramas, capturas de tela e fotos
   ```
   "O que há de errado com esta mensagem de erro?"
   "Pode explicar a tendência deste gráfico?"
   "Que espécie de planta é esta?"
   ```

2. Análise de documentos: extrair informações de PDFs, recibos e formulários
   ```
   "Resuma os pontos principais deste artigo científico"
   "Extraia os itens desta fatura"
   ```

3. Criação visual: gerar ou editar imagens a partir de descrições em texto
   ```
   "Crie um fluxograma mostrando como funciona a autenticação JWT"
   "Edite este logo para usar azul em vez de vermelho"
   ```

4. Raciocínio entre modalidades: conectar conceitos de formatos diferentes
   ```
   "Este código corresponde ao diagrama de arquitetura?"
   "Esta implementação de UI está consistente com o wireframe?"
   ```

### Exemplo real: depurando um componente React em 2 minutos

Na semana passada eu estava com um componente React que não renderizava direito. Depois de 15 minutos tentando de tudo, tirei uma captura de tela do código e da UI quebrada e mandei pro GPT-4V:

"O que está causando este problema de renderização? Aqui está o código do meu componente e a saída atual."

Em segundos, ele identificou o problema: eu estava desestruturando as props errado, o que deixava uma prop chave indefinida. Vinte minutos a mais de debug viraram dois.

### Como começar

Sem hardware especial, sem configuração complicada. Aqui está como chamar o GPT-4 com Vision via API:

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer SUA_CHAVE_API" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4-turbo",
    "messages": [
      {
        "role": "user",
        "content": [
          { "type": "text", "text": "Descreva esta imagem" },
          { "type": "image_url", "image_url": { "url": "https://example.com/image.png" } }
        ]
      }
    ]
  }'
```

Texto e imagem juntos, resposta detalhada do outro lado.

### Prompts para modelos multimodais: o que faz diferença

O jeito que você escreve o prompt importa mais aqui do que no texto puro. Três ajustes que funcionam:

1. Seja específico sobre o que quer analisar:
   ```
   Ruim: "O que você vê nesta captura de tela do painel?"
   Bom: "Nesta captura de tela do AWS CloudWatch, identifique quaisquer métricas com picos anormais nas últimas 24 horas"
   ```

2. Dê contexto para a imagem:
   ```
   Ruim: [Apenas imagem]
   Bom: "Este é um diagrama de rede da nossa infraestrutura AWS. Há algum problema de segurança?"
   ```

3. Peça saída estruturada:
   ```
   "Analise este gráfico de vendas e forneça: 
   1. A tendência geral
   2. Quaisquer anomalias
   3. Recomendações baseadas nos dados"
   ```

### Um uso que não esperava: acessibilidade

Descobri que dá pra usar IA multimodal para gerar texto alternativo e descrições de imagens, o que é útil em qualquer projeto que leve acessibilidade a sério:

```
# Gerar texto alternativo para imagens
"Crie um texto alt detalhado para esta imagem de produto que seria útil para usuários com deficiência visual"

# Converter informação visual em texto
"Descreva este diagrama arquitetônico em forma de texto que poderia ser lido por um leitor de tela"
```

Não é glamouroso, mas é um daqueles casos em que a ferramenta encaixa direto no problema.

### O que vem por aí

Compreensão de vídeo ainda é fraca na maioria dos modelos, mas está melhorando rápido. Análise em tempo real e modelos especializados para medicina e engenharia já aparecem nos papers de pesquisa.

Não precisa esperar por isso. Se você lida com código, imagens ou documentos, já tem o suficiente para experimentar agora.
