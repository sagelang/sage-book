# Installation

## Prerequisites

Sage requires a C linker and OpenSSL headers for compilation. Rust is **not** required.

**macOS:**
```bash
xcode-select --install
```

**Debian/Ubuntu:**
```bash
sudo apt install gcc libssl-dev
```

**Fedora/RHEL:**
```bash
sudo dnf install gcc openssl-devel
```

**Arch:**
```bash
sudo pacman -S gcc openssl
```

## Install Sage

### Homebrew (macOS)

```bash
brew install sagelang/sage/sage
```

### Quick Install (macOS/Linux)

```bash
curl -fsSL https://raw.githubusercontent.com/sagelang/sage/main/scripts/install.sh | bash
```

### Cargo (if you have Rust)

```bash
cargo install sage-lang
```

### Nix

```bash
nix profile install github:sagelang/sage
```

## Verify Installation

```bash
sage --version
```

You should see output like:

```
sage 2.0.2
```

## Next Steps

Now that Sage is installed, let's write your first program: [Hello World](./hello-world.md).
