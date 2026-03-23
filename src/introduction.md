# Introduction

Sage is a programming language where **agents are first-class citizens**.

Instead of building agents using Python frameworks like LangChain or CrewAI, you write agents as naturally as you write functions. Agents, their state, and their interactions are semantic primitives baked into the compiler and runtime.

```sage
agent Researcher {
    topic: String

    on start {
        let summary = try divine(
            "Write a concise 2-sentence summary of: {self.topic}"
        );
        yield(summary);
    }

    on error(e) {
        yield("Research unavailable");
    }
}

agent Coordinator {
    on start {
        let r1 = summon Researcher { topic: "quantum computing" };
        let r2 = summon Researcher { topic: "CRISPR gene editing" };

        let s1 = try await r1;
        let s2 = try await r2;

        print(s1);
        print(s2);
        yield(0);
    }

    on error(e) {
        print("A researcher failed");
        yield(1);
    }
}

run Coordinator;
```

## Why Sage?

**Agents as primitives, not patterns.** Most agent frameworks are libraries that impose patterns on top of a general-purpose language. Sage makes agents a first-class concept — the compiler understands what an agent is, what state it holds, and how agents communicate.

**Type-safe LLM integration.** The `divine` expression lets you call LLMs with structured output. The type system ensures you handle divination results correctly.

**Compiles to native binaries and WebAssembly.** Sage compiles to Rust, then to native code or WebAssembly. Your agent programs are fast, self-contained binaries — or run directly in the browser. Try it now in the [online playground](https://sagelang.github.io/sage-playground/).

**Concurrent by default.** Spawned agents run concurrently. The runtime handles scheduling and message passing.

**Built-in testing with LLM mocking.** Test your agents with deterministic mocks — no network calls, fast feedback, reliable CI.

## What You'll Learn

This guide covers:

1. **Getting Started** — Install Sage and write your first program
2. **Language Guide** — Syntax, types, and control flow
3. **Agents** — State, handlers, summoning, and messaging
4. **LLM Integration** — Using `divine` to call language models
5. **Tools** — Built-in tools like HTTP for external services
6. **Testing** — Write tests with first-class LLM mocking
7. **WebAssembly** — Compile agents for the browser and use the online playground
8. **Reference** — CLI commands, environment variables, error codes

Let's get started with [installation](./getting-started/installation.md).
