# Extern Functions (Rust FFI)

Sage can call Rust functions directly via `extern fn` declarations. This lets you drop into Rust for performance-critical code, system integration, or access to the Rust ecosystem.

## Declaring Extern Functions

Declare an extern function in Sage with the types it expects and returns:

```sage
extern fn now_iso() -> String
extern fn prompt(msg: String) -> String fails
extern fn clear_screen()
```

These declarations tell the compiler that the function is implemented in Rust and will be linked at compile time. You call them like any other Sage function:

```sage
let time = now_iso();
let input = try prompt("Enter your name:");
clear_screen();
```

## The `fails` Modifier

Functions marked `fails` can return errors. On the Rust side they return `Result<T, String>`, and in Sage they must be called with `try` or `catch`:

```sage
extern fn read_config(path: String) -> String fails

agent Main {
    on start {
        let config = try read_config("settings.toml");
        print(config);
        yield(0);
    }

    on error(e) {
        print("Failed to read config: " ++ e);
        yield(1);
    }
}

run Main;
```

## Implementing in Rust

Create a Rust source file (e.g., `src/sage_extern.rs`) with the function implementations:

```rust
// src/sage_extern.rs

pub fn now_iso() -> String {
    chrono::Utc::now().to_rfc3339()
}

pub fn prompt(msg: String) -> Result<String, String> {
    print!("{}", msg);
    std::io::Write::flush(&mut std::io::stdout())
        .map_err(|e| e.to_string())?;
    let mut input = String::new();
    std::io::stdin()
        .read_line(&mut input)
        .map_err(|e| e.to_string())?;
    Ok(input.trim().to_string())
}

pub fn clear_screen() {
    print!("\x1b[2J\x1b[H");
}
```

**Rules:**

- Each `extern fn` must have a corresponding `pub fn` in the Rust module
- Functions without `fails` return their type directly
- Functions with `fails` return `Result<T, String>`
- Functions returning nothing (`extern fn foo()`) map to `pub fn foo()` in Rust

## Type Mapping

| Sage Type | Rust Type |
|-----------|-----------|
| `String` | `String` |
| `Int` | `i64` |
| `Float` | `f64` |
| `Bool` | `bool` |
| `Unit` (no return) | `()` |

## Configuring grove.toml

Register your extern modules and any Cargo dependencies they need:

```toml
[project]
name = "my_project"
entry = "src/main.sg"

[extern]
modules = ["src/sage_extern.rs"]

[extern.dependencies]
chrono = "0.4"
reqwest = { version = "0.12", features = ["blocking"] }
```

- `modules` — list of Rust source files to compile and link
- `[extern.dependencies]` — additional Cargo dependencies needed by your extern code

The Sage compiler copies the extern modules into the generated Rust project and adds the dependencies to its Cargo.toml.

## Complete Example

**grove.toml:**
```toml
[project]
name = "greeter"
entry = "src/main.sg"

[extern]
modules = ["src/sage_extern.rs"]

[extern.dependencies]
chrono = "0.4"
```

**src/sage_extern.rs:**
```rust
pub fn now_iso() -> String {
    chrono::Utc::now().to_rfc3339()
}

pub fn styled(text: String, hex: String) -> String {
    let r = u8::from_str_radix(&hex[0..2], 16).unwrap_or(255);
    let g = u8::from_str_radix(&hex[2..4], 16).unwrap_or(255);
    let b = u8::from_str_radix(&hex[4..6], 16).unwrap_or(255);
    format!("\x1b[38;2;{};{};{}m{}\x1b[0m", r, g, b, text)
}
```

**src/main.sg:**
```sage
extern fn now_iso() -> String
extern fn styled(text: String, hex: String) -> String

agent Main {
    on start {
        let greeting = styled("Hello from Sage!", "4ECDC4");
        let time = now_iso();
        print(greeting);
        print("Current time: " ++ time);
        yield(0);
    }

    on error(e) {
        yield(1);
    }
}

run Main;
```

## When to Use Extern Functions

Extern functions are ideal for:

- **System integration** — terminal I/O, filesystem operations beyond the built-in `Fs` tool
- **Performance-critical code** — algorithms that benefit from direct Rust
- **Rust ecosystem access** — using any crate from crates.io
- **Custom tooling** — building domain-specific primitives for your agents

For most tasks, Sage's built-in tools (`Http`, `Database`, `Fs`, `Shell`) and standard library are sufficient. Use extern functions when you need something they don't cover.
