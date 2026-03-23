# Environment Variables

Sage uses environment variables to configure LLM integration and the compiler.

## LLM Configuration

These variables configure the `divine` expression.

### SAGE_API_KEY

**Required** for LLM features. Your API key for the LLM provider.

```bash
export SAGE_API_KEY="sk-..."
```

### SAGE_LLM_URL

Base URL for the LLM API. Defaults to OpenAI.

```bash
# OpenAI (default)
export SAGE_LLM_URL="https://api.openai.com/v1"

# Ollama (local)
export SAGE_LLM_URL="http://localhost:11434/v1"

# Azure OpenAI
export SAGE_LLM_URL="https://your-resource.openai.azure.com/openai/deployments/your-deployment"

# Other OpenAI-compatible providers
export SAGE_LLM_URL="https://api.together.xyz/v1"
```

### SAGE_MODEL

Which model to use. Default: `gpt-4o-mini`

```bash
export SAGE_MODEL="gpt-4o"
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

### SAGE_INFER_RETRIES

Maximum retries for structured output parsing. When `divine` returns a type other than `String`, the runtime parses the LLM's response as JSON. If parsing fails, it retries with error feedback. Default: `3`

```bash
export SAGE_INFER_RETRIES="5"
```

## Tool Configuration

These variables configure built-in tools used by agents.

### SAGE_HTTP_TIMEOUT

HTTP request timeout in seconds. Default: `30`

```bash
export SAGE_HTTP_TIMEOUT="60"
```

### SAGE_DATABASE_URL

Database connection URL for the `Database` tool. Required when using database features.

```bash
# SQLite
export SAGE_DATABASE_URL="sqlite:./data.db"
export SAGE_DATABASE_URL="sqlite::memory:"

# PostgreSQL
export SAGE_DATABASE_URL="postgres://user:password@localhost/dbname"

# MySQL
export SAGE_DATABASE_URL="mysql://user:password@localhost/dbname"
```

### SAGE_FS_ROOT

Root directory for `Fs` tool operations. All file paths are relative to this directory. Default: `.` (current directory)

```bash
export SAGE_FS_ROOT="/var/data/myapp"
```

## Observability

### SAGE_TRACE

Enable trace output to stderr. Set to `1` to enable.

```bash
export SAGE_TRACE=1
```

### SAGE_TRACE_FILE

Write trace output to a file instead of stderr.

```bash
export SAGE_TRACE_FILE="trace.log"
```

## Compiler Configuration

### SAGE_TOOLCHAIN

Override the path to the pre-compiled toolchain. Normally this is detected automatically.

```bash
export SAGE_TOOLCHAIN="/path/to/toolchain"
```

The toolchain directory should contain:
- `bin/rustc` - The Rust compiler
- `libs/` - Pre-compiled runtime libraries

## Using .env Files

Sage automatically loads `.env` files from the current directory:

```bash
# .env
SAGE_API_KEY=sk-...
SAGE_MODEL=gpt-4o
SAGE_MAX_TOKENS=2048
```

This is useful for per-project configuration and keeping secrets out of your shell history.

## Provider Quick Reference

### OpenAI

```bash
export SAGE_API_KEY="sk-..."
export SAGE_MODEL="gpt-4o"
```

### Ollama (Local)

```bash
export SAGE_LLM_URL="http://localhost:11434/v1"
export SAGE_MODEL="llama2"
# No API key needed
```

### Azure OpenAI

```bash
export SAGE_LLM_URL="https://your-resource.openai.azure.com/openai/deployments/your-deployment"
export SAGE_API_KEY="your-azure-key"
export SAGE_MODEL="gpt-4"
```

### Together AI

```bash
export SAGE_LLM_URL="https://api.together.xyz/v1"
export SAGE_API_KEY="your-key"
export SAGE_MODEL="meta-llama/Llama-3-70b-chat-hf"
```
