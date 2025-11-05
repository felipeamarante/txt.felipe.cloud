+++
title = "LLMs Locais: Execute Modelos de IA no Seu Laptop Sem Internet"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-03-14"
categories = [
    "AI",
]
menu = "main"
+++

### Cansado de pagar por chamadas de API e compartilhar seu código com a nuvem?

![local ai](/images/local-ai.gif)

Todos já passamos por isso: atingindo limites de taxa na API da OpenAI, preocupados com código sensível sendo enviado para algum servidor distante, ou tentando programar em um avião com Wi-Fi instável. E se você pudesse executar modelos de IA poderosos diretamente no seu laptop, completamente offline?

### Conheça o Ollama: LLMs Locais de Forma Ridiculamente Simples

O Ollama permite executar grandes modelos de linguagem localmente com configuração mínima. Sem contas na nuvem, sem chaves de API, sem limites de uso.

```bash
# Instale
brew install ollama
```

### Comece a usar em 30 segundos

```bash
# Inicie o serviço
ollama serve

# Em outro terminal, baixe um modelo
ollama pull llama2

# Comece a conversar
ollama run llama2
```

É isso. Você agora está executando um poderoso LLM completamente offline.

### Escolha o modelo certo para suas necessidades

Diferentes tarefas precisam de diferentes modelos. Aqui está minha tabela de referência:

```bash
# Para ajuda com programação (6.7B parâmetros, rápido na maioria dos laptops)
ollama pull codellama:7b

# Para assistência geral (equilíbrio entre tamanho/qualidade)
ollama pull llama2:13b

# Para raciocínio mais poderoso (precisa de GPU decente)
ollama pull mistral

# Para modelo pequeno que roda em qualquer hardware
ollama pull tinyllama
```

### Integre com seu fluxo de trabalho

A verdadeira mágica acontece quando você integra o Ollama às suas ferramentas de desenvolvimento:

**VS Code**: Instale a extensão "Continue" e aponte para seu endpoint Ollama local.

**Terminal**: Crie esta simples função shell:

```bash
function perguntar() {
  curl -s "http://localhost:11434/api/generate" \
    -d "{\"model\": \"codellama\", \"prompt\": \"$1\", \"stream\": false}" | 
    jq -r '.response'
}
```

Agora você pode simplesmente digitar:

```bash
perguntar "Escreva uma função bash para encontrar os maiores arquivos em um diretório"
```

**Git hooks**: Crie um hook pre-commit que usa o Ollama para revisar suas alterações de código:

```bash
#!/bin/bash
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(js|jsx|ts|tsx)$')

if [[ "$STAGED_FILES" = "" ]]; then
  exit 0
fi

echo "Executando revisão de código com IA..."
for FILE in $STAGED_FILES; do
  DIFF=$(git diff --cached "$FILE")
  REVIEW=$(curl -s "http://localhost:11434/api/generate" \
    -d "{\"model\": \"codellama\", \"prompt\": \"Revise esta alteração de código e sugira melhorias: $DIFF\", \"stream\": false}" | 
    jq -r '.response')
  
  echo -e "\n\033[1;34mRevisão de $FILE:\033[0m\n$REVIEW\n"
done
```

### Dicas de desempenho que realmente funcionam

Executar LLMs localmente pode consumir muitos recursos, mas estas dicas fazem uma grande diferença:

1. **Modelos quantizados**: Use versões quantizadas de 4 bits para um aumento de 2-3x na velocidade
   ```bash
   ollama pull codellama:7b-q4_0
   ```

2. **Ajuste a janela de contexto**: Contexto menor = respostas mais rápidas
   ```bash
   ollama run llama2 --context 2048
   ```

3. **Aceleração de GPU**: Se você tem uma GPU NVIDIA/AMD decente, o Ollama a usa automaticamente

4. **Remova modelos não utilizados**: Eles ocupam espaço significativo em disco
   ```bash
   ollama list
   ollama rm modelo-nao-usado
   ```

### O caso de uso do mundo real

Na semana passada, eu estava trabalhando em um projeto de cliente com requisitos de segurança rigorosos: nenhum código poderia sair da rede deles. Com o Ollama rodando localmente, eu ainda conseguia obter assistência de IA para depurar um fluxo de autenticação complexo sem violar nenhuma política.

O modelo me ajudou a identificar uma condição de corrida na lógica de atualização de token que teria levado horas para encontrar manualmente. Tudo isso sem enviar uma única linha de código para uma API externa.

### Além do básico: Crie seus próprios modelos personalizados

O recurso mais poderoso é criar modelos personalizados adaptados às suas necessidades:

```
FROM llama2
SYSTEM Você é um assistente especializado em infraestrutura AWS.

# Inclua documentação AWS
PARAMETER temperature 0.7
PARAMETER num_ctx 4096
```

Salve isso como `Modelfile` e construa:

```bash
ollama create aws-assistant -f Modelfile
ollama run aws-assistant
```

Agora você tem um assistente de IA especializado em AWS, rodando completamente offline.

### Sem mais desculpas para não usar IA no seu fluxo de trabalho

Com o Ollama, você obtém:
- Privacidade completa
- Sem custos de API
- Acesso offline
- Modelos personalizáveis

A melhor parte? É tudo open source. Experimente e me conte como isso transforma seu fluxo de trabalho de desenvolvimento!
