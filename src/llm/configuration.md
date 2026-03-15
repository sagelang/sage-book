# Configuration

Configure LLM behavior through environment variables.

## Required

### SAGE_API_KEY

Your OpenAI API key (or compatible provider):

```bash
export SAGE_API_KEY="sk-..."
```

Or in a `.env` file in your project directory:

```
SAGE_API_KEY=sk-...
```

## Optional

### SAGE_LLM_URL

Base URL for the LLM API. Defaults to OpenAI:

```bash
export SAGE_LLM_URL="https://api.openai.com/v1"
```

For local models (Ollama):

```bash
export SAGE_LLM_URL="http://localhost:11434/v1"
```

For other providers (Azure, Anthropic-compatible, etc.):

```bash
export SAGE_LLM_URL="https://your-provider.com/v1"
```

### SAGE_MODEL

Which model to use. Default: `gpt-4o-mini`

```bash
export SAGE_MODEL="gpt-4o"
```

For Ollama:

```bash
export SAGE_MODEL="llama2"
```

### SAGE_MAX_TOKENS

Maximum tokens per response. Default: `1024`

```bash
export SAGE_MAX_TOKENS="2048"
```

### SAGE_TIMEOUT_MS

Request timeout in milliseconds. Default: `30000` (30 seconds)

```bash
export SAGE_TIMEOUT_MS="60000"
```

## Using .env Files

Sage automatically loads `.env` files from the current directory:

```
# .env
SAGE_API_KEY=sk-...
SAGE_MODEL=gpt-4o
SAGE_MAX_TOKENS=2048
```

## Provider Examples

### OpenAI (default)

```bash
export SAGE_API_KEY="sk-..."
# SAGE_LLM_URL defaults to OpenAI
export SAGE_MODEL="gpt-4o"
```

### Ollama (local)

```bash
export SAGE_LLM_URL="http://localhost:11434/v1"
export SAGE_MODEL="llama2"
# No API key needed for local Ollama
```

### Azure OpenAI

```bash
export SAGE_LLM_URL="https://your-resource.openai.azure.com/openai/deployments/your-deployment"
export SAGE_API_KEY="your-azure-key"
export SAGE_MODEL="gpt-4"
```

### Other OpenAI-Compatible Providers

Any provider with an OpenAI-compatible API should work:

```bash
export SAGE_LLM_URL="https://api.together.xyz/v1"
export SAGE_API_KEY="your-key"
export SAGE_MODEL="meta-llama/Llama-3-70b-chat-hf"
```

## Troubleshooting

### "API key not set"

Make sure `SAGE_API_KEY` is exported or in your `.env` file.

### Timeout errors

Increase `SAGE_TIMEOUT_MS` for slow models or complex prompts.

### Connection refused

Check `SAGE_LLM_URL` is correct and the service is running.
