# Zed

[Zed](https://zed.dev) is a high-performance code editor with native Sage support.

## Installation

1. Open Zed
2. Press `Cmd+Shift+X` to open Extensions
3. Search for "Sage"
4. Click Install

Alternatively, use the command palette (`Cmd+Shift+P`) and run "zed: install extension".

## Features

The Sage extension for Zed provides:

- **Tree-sitter Highlighting** — Fast, accurate syntax highlighting
- **LSP Diagnostics** — Real-time error reporting from the Sage compiler
- **Auto-indentation** — Smart indentation for agents, functions, and blocks

## Requirements

The language server requires `sage` to be on your PATH. Install via:

```bash
# Homebrew (macOS)
brew install sagelang/sage/sage

# Cargo
cargo install sage-lang

# Quick install
curl -fsSL https://raw.githubusercontent.com/sagelang/sage/main/scripts/install.sh | bash
```

## Troubleshooting

### No syntax highlighting

If syntax highlighting isn't working:

1. Ensure the file has a `.sg` extension
2. Check that the Sage extension is installed (Extensions → Installed)
3. Try restarting Zed

### No diagnostics

If you're not seeing error diagnostics:

1. Verify `sage` is on your PATH: `which sage`
2. Check Zed logs: `Cmd+Shift+P` → "zed: open log"
3. Look for "sage-sense" or "language server" errors

### Extension not loading

If the extension fails to load:

1. Uninstall the extension
2. Restart Zed
3. Reinstall the extension
