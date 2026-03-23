# WASM Target

Sage can compile agents to WebAssembly for browser execution.

## Building for WASM

```bash
sage build hello.sg --target web
```

This compiles your Sage program through the full pipeline (parse, type-check, codegen) but targets `wasm32-unknown-unknown` instead of your native platform.

### Output

The build produces a `pkg/` directory containing:

```
pkg/
  hello.js          # JavaScript glue (wasm-bindgen)
  hello_bg.wasm     # WebAssembly binary
```

### Prerequisites

The WASM target requires:

- `wasm32-unknown-unknown` Rust target: `rustup target add wasm32-unknown-unknown`
- `wasm-bindgen-cli`: `cargo install wasm-bindgen-cli`
- (Optional) `wasm-opt` for size optimisation: install via [binaryen](https://github.com/WebAssembly/binaryen)

### Target values

The `--target` flag accepts:

| Value | Description |
|-------|-------------|
| `native` | Default. Compile to a native binary. |
| `web` or `wasm` | Compile to WebAssembly for browser use. |

## How It Works

When targeting WASM, the codegen layer:

1. Generates a `#[wasm_bindgen(start)]` entry point instead of `#[tokio::main]`
2. Uses `sage-runtime-web` — a browser-compatible runtime that replaces `tokio`, `reqwest`, and native I/O with Web APIs
3. Produces a `cdylib` crate compiled with `wasm-bindgen`
4. Optionally runs `wasm-opt -Oz` for size optimisation

## Using in a Web Page

```html
<script type="module">
  import init from './pkg/hello.js';
  await init();
</script>
```

The WASM module initialises automatically via the `#[wasm_bindgen(start)]` entry point.

## Limitations

- `divine` (LLM calls) requires a browser-accessible OpenAI-compatible endpoint
- `Database` and `Shell` tools are not available in WASM
- `Fs` operations use browser storage APIs instead of the filesystem
- Agent concurrency uses `wasm_bindgen_futures::spawn_local` (single-threaded)
