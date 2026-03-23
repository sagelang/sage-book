# Online Playground

Try Sage instantly in your browser — no installation required.

**[sagelang.github.io/sage-playground](https://sagelang.github.io/sage-playground/)**

## Features

- **Editable code** — write any Sage code in the browser
- **Live output** — see `print()` output and yield values immediately
- **Syntax highlighting** — keywords, types, strings, numbers, builtins, and comments
- **Example programs** — Hello World, Counter Loop, String Operations, Fibonacci
- **Keyboard shortcuts** — Ctrl/Cmd+Enter to run, Tab to indent

## How It Works

The playground uses a tree-walking interpreter (`sage-playground-engine`) compiled to WebAssembly. It shares the same parser as the full Sage compiler but interprets the AST directly instead of generating Rust code.

The interpreter supports:

- Variables, assignments, and scoping
- Functions (user-defined and standard library)
- Control flow: `if`/`else`, `while`, `for`, `loop`, `break`, `return`
- Records, enums, tuples, and pattern matching
- String operations and interpolation
- `print()`, `trace()`, and `yield()`
- Infinite loop protection (1M step limit)

## What's Not Supported

The playground interpreter does not support features that require external services:

- `divine` / `infer` (LLM calls)
- Tool calls (`Http`, `Database`, `Fs`, `Shell`)
- Agent spawning (`summon`, `await`)
- Supervisors and protocols
- Persistence (`@persistent`)

These features work in the full compiled Sage — the playground focuses on exploring the core language.
