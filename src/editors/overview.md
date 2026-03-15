# Editor Support

Sage includes first-class editor support with syntax highlighting and real-time diagnostics via the Language Server Protocol (LSP).

## Supported Editors

| Editor | Extension | Highlighting | Diagnostics |
|--------|-----------|--------------|-------------|
| [Zed](https://zed.dev) | Built-in | Tree-sitter | LSP |
| [VS Code](https://code.visualstudio.com) | Marketplace | TextMate | LSP |

## Features

All Sage editor extensions provide:

- **Syntax Highlighting** — Keywords, strings, comments, types, and more
- **Real-time Diagnostics** — Errors and warnings as you type
- **Auto-indentation** — Smart indentation for blocks and expressions

## Language Server

The Sage language server (`sage-sense`) provides:

- Parse error reporting
- Type checking errors
- Undefined variable detection
- All compiler diagnostics in real-time

The language server is built into the `sage` CLI and starts automatically when you open a `.sg` file in a supported editor.

## Manual LSP Setup

If you're using an editor that supports LSP but doesn't have a Sage extension, you can configure it to use:

```bash
sage sense
```

This starts the language server on stdin/stdout using the standard LSP protocol.
