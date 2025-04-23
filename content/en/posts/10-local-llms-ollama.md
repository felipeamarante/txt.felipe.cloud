+++
title = "Local LLMs: Run AI Models on Your Laptop Without Internet"
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

### Tired of paying for API calls and sharing your code with the cloud?

![local ai](/images/local-ai.gif)

We've all been there—hitting rate limits on OpenAI's API, worrying about sensitive code being uploaded to some distant server, or trying to code on a plane with spotty Wi-Fi. What if you could run powerful AI models right on your laptop, completely offline?

### Enter Ollama: Local LLMs Made Ridiculously Simple

Ollama lets you run large language models locally with minimal setup. No cloud accounts, no API keys, no usage limits.

```bash
# Install it
brew install ollama
```

### Get up and running in 30 seconds

```bash
# Start the service
ollama serve

# In another terminal, pull a model
ollama pull llama2

# Start chatting
ollama run llama2
```

That's it. You're now running a powerful LLM completely offline.

### Choose the right model for your needs

Different tasks need different models. Here's my cheat sheet:

```bash
# For coding help (6.7B parameters, fast on most laptops)
ollama pull codellama:7b

# For general assistance (balanced size/quality)
ollama pull llama2:13b

# For more powerful reasoning (needs decent GPU)
ollama pull mistral

# For tiny model that runs on anything
ollama pull tinyllama
```

### Integrate with your workflow

The real magic happens when you integrate Ollama into your development tools:

**VS Code**: Install "Continue" extension and point it to your local Ollama endpoint.

**Terminal**: Create this simple shell function:

```bash
function ask() {
  curl -s "http://localhost:11434/api/generate" \
    -d "{\"model\": \"codellama\", \"prompt\": \"$1\", \"stream\": false}" | 
    jq -r '.response'
}
```

Now you can just type:

```bash
ask "Write a bash function to find the largest files in a directory"
```

**Git hooks**: Create a pre-commit hook that uses Ollama to review your code changes:

```bash
#!/bin/bash
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(js|jsx|ts|tsx)$')

if [[ "$STAGED_FILES" = "" ]]; then
  exit 0
fi

echo "Running AI code review..."
for FILE in $STAGED_FILES; do
  DIFF=$(git diff --cached "$FILE")
  REVIEW=$(curl -s "http://localhost:11434/api/generate" \
    -d "{\"model\": \"codellama\", \"prompt\": \"Review this code change and suggest improvements: $DIFF\", \"stream\": false}" | 
    jq -r '.response')
  
  echo -e "\n\033[1;34m$FILE Review:\033[0m\n$REVIEW\n"
done
```

### Performance tips that actually work

Running LLMs locally can be resource-intensive, but these tips make a huge difference:

1. **Quantized models**: Use 4-bit quantized versions for 2-3x speedup
   ```bash
   ollama pull codellama:7b-q4_0
   ```

2. **Adjust context window**: Smaller context = faster responses
   ```bash
   ollama run llama2 --context 2048
   ```

3. **GPU acceleration**: If you have a decent NVIDIA/AMD GPU, Ollama uses it automatically

4. **Prune unused models**: They take up significant disk space
   ```bash
   ollama list
   ollama rm unused-model
   ```

### The real-world use case

Last week, I was working on a client project with strict security requirements—no code could leave their network. With Ollama running locally, I could still get AI assistance for debugging a complex authentication flow without violating any policies.

The model helped me identify a race condition in the token refresh logic that would have taken hours to find manually. All without sending a single line of code to an external API.

### Beyond the basics: Create your own custom models

The most powerful feature is creating custom models tailored to your needs:

```
FROM llama2
SYSTEM You are a specialized assistant for AWS infrastructure.

# Include AWS documentation
PARAMETER temperature 0.7
PARAMETER num_ctx 4096
```

Save this as `Modelfile` and build it:

```bash
ollama create aws-assistant -f Modelfile
ollama run aws-assistant
```

Now you have an AI assistant specialized in AWS, running completely offline.

### No more excuses for not using AI in your workflow

With Ollama, you get:
- Complete privacy
- No API costs
- Offline access
- Customizable models

The best part? It's all open source. Give it a try and let me know how it transforms your development workflow!
