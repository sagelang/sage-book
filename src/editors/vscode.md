# VS Code

[Visual Studio Code](https://code.visualstudio.com) is supported via the Sage extension.

## Installation

1. Open VS Code
2. Press `Cmd+Shift+X` (Mac) or `Ctrl+Shift+X` (Windows/Linux) to open Extensions
3. Search for "Sage"
4. Click Install

## Features

The Sage extension for VS Code provides:

- **TextMate Highlighting** — Syntax highlighting for all Sage constructs
- **LSP Diagnostics** — Real-time error reporting from the Sage compiler
- **File Icons** — Custom icon for `.sg` files

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

## Configuration

The extension can be configured in VS Code settings:

```json
{
  "sage.path": "/usr/local/bin/sage"
}
```

| Setting | Description | Default |
|---------|-------------|---------|
| `sage.path` | Path to the `sage` binary | Auto-detected from PATH |

## Troubleshooting

### No syntax highlighting

If syntax highlighting isn't working:

1. Ensure the file has a `.sg` extension
2. Check that the Sage extension is installed
3. Try reloading the window: `Cmd+Shift+P` → "Developer: Reload Window"

### No diagnostics

If you're not seeing error diagnostics:

1. Verify `sage` is on your PATH: `which sage`
2. Check the Output panel: View → Output → select "Sage Language Server"
3. Look for connection or startup errors

### Extension not activating

If the extension isn't activating:

1. Check the Extensions panel for errors
2. Disable and re-enable the extension
3. Check VS Code's developer console for errors
