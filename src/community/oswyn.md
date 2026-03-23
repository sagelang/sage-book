# Oswyn — Your Sage Companion

Oswyn is a browser-based AI assistant that knows everything about the Sage programming language. Ask questions about syntax, agents, LLM integration, tools, testing, supervision trees, and more — and get working code examples in return.

**[Chat with Oswyn →](https://sagelang.github.io/oswyn/)**

## Features

- Ask questions about any part of Sage and get working code examples
- Conversation memory across sessions (stored in your browser's localStorage)
- Supports OpenAI, Anthropic, Ollama, and any OpenAI-compatible API
- Runs entirely in your browser — no backend, no data collection
- Your API key never leaves your browser

## Setup

1. Open [sagelang.github.io/oswyn](https://sagelang.github.io/oswyn/)
2. Click the settings icon and enter your API key
3. Start chatting

## Example Questions

- "How do I create my first Sage agent?"
- "Explain the `divine` expression and LLM integration"
- "How do supervision trees work in Sage?"
- "Show me how to use the Http tool"
- "How do I mock LLM calls in tests?"

## Privacy

Oswyn runs entirely in your browser. Your API key and conversation history are stored in localStorage and are never sent to any server other than your chosen LLM provider.

## Oswyn in the CLI

You'll also encounter Oswyn in the Sage CLI. When you compile and run programs, Oswyn provides warm, encouraging feedback:

```
👻 Oswyn is pleased. Your program compiled successfully.
→ sage run examples/research.sg

👻 Oswyn is consulting the ancient texts...
→ divine() awaiting LLM response
```

Ward handles the stern compiler warnings. Oswyn handles the encouragement. Between them, you're in good hands.
