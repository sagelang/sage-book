# Introduction

Sage is a programming language where **agents are first-class citizens**.

Instead of building agents using Python frameworks like LangChain or CrewAI, you write agents as naturally as you write functions. Agents, their state, and their interactions are semantic primitives baked into the compiler and runtime.

```sage
agent Researcher {
    topic: String

    on start {
        let summary = try infer(
            "Write a concise 2-sentence summary of: {self.topic}"
        );
        emit(summary);
    }

    on error(e) {
        emit("Research unavailable");
    }
}

agent Coordinator {
    on start {
        let r1 = spawn Researcher { topic: "quantum computing" };
        let r2 = spawn Researcher { topic: "CRISPR gene editing" };

        let s1 = try await r1;
        let s2 = try await r2;

        print(s1);
        print(s2);
        emit(0);
    }

    on error(e) {
        print("A researcher failed");
        emit(1);
    }
}

run Coordinator;
```

## Why Sage?

**Agents as primitives, not patterns.** Most agent frameworks are libraries that impose patterns on top of a general-purpose language. Sage makes agents a first-class concept — the compiler understands what an agent is, what state it holds, and how agents communicate.

**Type-safe LLM integration.** The `infer` expression lets you call LLMs with structured output. The type system ensures you handle inference results correctly.

**Compiles to native binaries.** Sage compiles to Rust, then to native code. Your agent programs are fast, self-contained binaries with no runtime dependencies.

**Concurrent by default.** Spawned agents run concurrently. The runtime handles scheduling and message passing.

**Built-in testing with LLM mocking.** Test your agents with deterministic mocks — no network calls, fast feedback, reliable CI.

## What You'll Learn

This guide covers:

1. **Getting Started** — Install Sage and write your first program
2. **Language Guide** — Syntax, types, and control flow
3. **Agents** — State, handlers, spawning, and messaging
4. **LLM Integration** — Using `infer` to call language models
5. **Tools** — Built-in tools like HTTP for external services
6. **Testing** — Write tests with first-class LLM mocking
7. **Reference** — CLI commands, environment variables, error codes

Let's get started with [installation](./getting-started/installation.md).
