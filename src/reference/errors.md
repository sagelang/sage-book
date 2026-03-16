# Error Messages

Sage provides helpful error messages with source locations and suggestions.

## Parse Errors

### Unexpected token

```
error: unexpected token
  --> hello.sg:5:10
  |
5 |     let x =
  |          ^ expected expression
```

**Fix**: Complete the expression or remove the incomplete statement.

### Missing semicolon

```
error: expected ';'
  --> hello.sg:3:15
  |
3 |     let x = 42
  |               ^ expected ';' after statement
```

**Fix**: Add a semicolon at the end of the statement.

### Unclosed brace

```
error: unclosed '{'
  --> hello.sg:2:12
  |
2 |     on start {
  |              ^ this '{' was never closed
```

**Fix**: Add the matching closing brace `}`.

## Type Errors

### Type mismatch

```
error: type mismatch
  --> hello.sg:7:20
  |
7 |     let x: Int = "hello";
  |                  ^^^^^^^ expected Int, found String
```

**Fix**: Use a value of the correct type or change the type annotation.

### Undefined variable

```
error: undefined variable 'foo'
  --> hello.sg:5:10
  |
5 |     print(foo);
  |           ^^^ not found in this scope
```

**Fix**: Define the variable before using it, or check for typos.

### Unknown agent

```
error: unknown agent 'Worker'
  --> hello.sg:10:22
  |
10 |     let w = summon Worker {};
   |                   ^^^^^^ agent not defined
```

**Fix**: Define the agent or check the spelling.

### Missing field

```
error: missing field 'name'
  --> hello.sg:15:22
  |
15 |     let g = summon Greeter {};
   |                   ^^^^^^^^^ field 'name' not provided
```

**Fix**: Provide all required fields when spawning:

```sage
let g = summon Greeter { name: "World" };
```

### Unhandled fallible operation (E013)

```
error[E013]: fallible operation must be handled
  --> hello.sg:5:15
  |
5 |     let x = divine("prompt");
  |             ^^^^^^^^^^^^^^^ this can fail
  |
  = help: use 'try' to propagate or 'catch' to handle inline
```

**Fix**: Handle the error with `try` or `catch`:

```sage
// Propagate to on error handler
let x = try divine("prompt");

// Or handle inline
let x = catch divine("prompt") {
    "fallback"
};
```

### Wrong message type

```
error: type mismatch in send
  --> hello.sg:8:10
  |
8 |     try send(worker, "hello");
  |              ^^^^^^^^^^^^^^^^ worker expects WorkerMsg, got String
```

**Fix**: Send a value of the type the agent accepts (defined by its `receives` clause).

## Runtime Errors

### API key not set

```
error: SAGE_API_KEY environment variable not set
```

**Fix**: Set your API key:

```bash
export SAGE_API_KEY="sk-..."
```

### LLM timeout

```
error: LLM request timed out after 30000ms
```

**Fix**: Increase the timeout or use a faster model:

```bash
export SAGE_TIMEOUT_MS="60000"
```

### Connection refused

```
error: failed to connect to LLM API
```

**Fix**: Check that `SAGE_LLM_URL` is correct and the service is running.

## Compilation Errors

### Rust not found (cargo mode)

```
error: Failed to run cargo build. Is Rust installed?
```

This happens when using the cargo fallback without Rust installed.

**Fix**: Either:
- Install Sage using the install script (includes pre-compiled toolchain)
- Install Rust from https://rustup.rs

### Linker not found

```
error: linker 'cc' not found
```

**Fix**: Install a C compiler:

```bash
# Ubuntu/Debian
sudo apt install gcc

# macOS
xcode-select --install
```

## Getting Help

If you encounter an error not listed here:

1. Check the [GitHub issues](https://github.com/sagelang/sage/issues)
2. Open a new issue with:
   - The error message
   - Your Sage code (minimal example)
   - Your environment (OS, Sage version)
