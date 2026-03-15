# CLI Commands

The `sage` command-line tool compiles and runs Sage programs.

## sage new

Create a new Sage project with scaffolding:

```bash
sage new my_project
```

This creates:

```
my_project/
├── sage.toml           # Project manifest
└── src/
    └── main.sg         # Entry point with example code
```

### Examples

```bash
# Create a new project
sage new my_agent

# Enter the project and run it
cd my_agent
sage run .
```

## sage run

Compile and execute a Sage program:

```bash
sage run program.sg
```

### Options

| Option | Description |
|--------|-------------|
| `--release` | Build with optimizations |
| `-q, --quiet` | Minimal output |

### Examples

```bash
# Run a program
sage run hello.sg

# Run with optimizations
sage run hello.sg --release

# Run quietly (only program output)
sage run hello.sg -q
```

## sage build

Compile a Sage program to a native binary without running it:

```bash
sage build program.sg
```

### Options

| Option | Description |
|--------|-------------|
| `--release` | Build with optimizations |
| `-o, --output <dir>` | Output directory (default: `target/sage`) |
| `--emit-rust` | Only generate Rust code, don't compile |

### Examples

```bash
# Build a binary
sage build hello.sg

# Build with optimizations
sage build hello.sg --release

# Custom output directory
sage build hello.sg -o ./out

# Generate Rust code only (for inspection)
sage build hello.sg --emit-rust
```

### Output Structure

After building, you'll find:

```
target/sage/
  hello/
    main.rs      # Generated Rust code
    hello        # Native binary (if not --emit-rust)
```

## sage check

Type-check a Sage program without compiling or running:

```bash
sage check program.sg
```

This is useful for quick validation during development.

### Examples

```bash
# Check for errors
sage check hello.sg

# Output on success:
# ✨ No errors in hello.sg
```

## sage test

Run tests in a Sage project:

```bash
sage test .
```

This discovers all `*_test.sg` files, compiles them, and runs the tests.

### Options

| Option | Description |
|--------|-------------|
| `--filter <pattern>` | Only run tests matching the pattern |
| `--file <path>` | Run only tests in the specified file |
| `--serial` | Run all tests sequentially (not in parallel) |
| `-v, --verbose` | Show detailed failure output |
| `--no-colour` | Disable colored output |

### Examples

```bash
# Run all tests in the project
sage test .

# Run tests matching "auth"
sage test . --filter auth

# Run tests in a specific file
sage test . --file src/utils_test.sg

# Run tests sequentially (useful for debugging)
sage test . --serial

# Verbose output with failure details
sage test . --verbose
```

### Output

```
🦉 Ward Running 3 tests from 2 files

  PASS auth_test.sg::login succeeds with valid credentials
  PASS auth_test.sg::login fails with invalid password
  FAIL utils_test.sg::parse handles empty input

🦉 Ward test result: FAILED. 2 passed, 1 failed, 0 skipped [1.23s]
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All tests passed |
| 1 | One or more tests failed |

## sage sense

Start the Language Server Protocol (LSP) server for editor integration:

```bash
sage sense
```

This command starts the Sage language server on stdin/stdout. It's typically invoked automatically by editor extensions (Zed, VS Code) rather than manually.

### Features

The language server provides:

- Real-time parse error reporting
- Type checking diagnostics
- Undefined variable detection
- All compiler error codes

### Manual Usage

For editors without a Sage extension, configure the LSP client to run `sage sense` as the language server command.

Example for generic LSP configuration:

```json
{
  "languageId": "sage",
  "command": "sage",
  "args": ["sense"],
  "fileExtensions": [".sg"]
}
```

## Global Options

| Option | Description |
|--------|-------------|
| `-h, --help` | Show help information |
| `-V, --version` | Show version |

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Compilation error (parse, type, or codegen) |
| Other | Program exit code (when using `sage run`) |

## Compilation Modes

Sage automatically selects the fastest compilation mode:

### Pre-compiled Toolchain (Default)

When installed via the install script or release binaries, Sage includes a pre-compiled Rust toolchain. This provides fast compilation without requiring Rust to be installed.

### Cargo Fallback

If no pre-compiled toolchain is found, Sage falls back to using `cargo`. This requires Rust to be installed but allows compilation on any platform.

The output will indicate which mode was used:

```
✨ Done Compiled hello.sg in 0.42s           # Pre-compiled toolchain
✨ Done Compiled hello.sg (cargo) in 2.31s   # Cargo fallback
```
