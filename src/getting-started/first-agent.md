# Your First Agent

Let's build an agent that does something useful — fetching information from an LLM.

## Setup

First, set your OpenAI API key:

```bash
export SAGE_API_KEY="your-openai-api-key"
```

Or create a `.env` file in your project directory:

```
SAGE_API_KEY=your-openai-api-key
```

## The Program

Create `researcher.sg`:

```sage
agent Researcher {
    topic: String

    on start {
        let summary = try divine(
            "Write a concise 2-sentence summary of: {self.topic}"
        );
        print(summary);
        yield(summary);
    }

    on error(e) {
        yield("Research failed");
    }
}

agent Main {
    on start {
        let r = summon Researcher { topic: "the Rust programming language" };
        let result = try await r;
        print("Research complete!");
        yield(0);
    }

    on error(e) {
        print("Something went wrong");
        yield(1);
    }
}

run Main;
```

## Run It

```bash
sage run researcher.sg
```

Output (will vary based on LLM response):

```
Rust is a systems programming language focused on safety, concurrency, and performance. It achieves memory safety without garbage collection through its ownership system.
Research complete!
0
```

## What's Happening?

1. **`topic: String`** — The `Researcher` agent has a field called `topic`. Fields are the agent's state, initialized when spawned.

2. **`try divine("...")`** — Calls the LLM with the given prompt. The `{self.topic}` syntax interpolates the agent's field into the prompt. The `try` propagates errors to `on error`.

3. **`on error(e)`** — Handles errors from `try` expressions. Without this, the agent would panic on failure.

4. **`summon Researcher { topic: "..." }`** — Creates a new `Researcher` agent with the given field value.

5. **`try await r`** — Waits for the agent to yield its result. The summoned agent runs concurrently until awaited.

## Multiple Agents

Let's summon multiple researchers in parallel:

```sage
agent Researcher {
    topic: String

    on start {
        let summary = try divine(
            "One sentence about: {self.topic}"
        );
        yield(summary);
    }

    on error(e) {
        yield("Research unavailable");
    }
}

agent Main {
    on start {
        let r1 = summon Researcher { topic: "quantum computing" };
        let r2 = summon Researcher { topic: "machine learning" };
        let r3 = summon Researcher { topic: "blockchain" };

        // All three run concurrently
        let s1 = try await r1;
        let s2 = try await r2;
        let s3 = try await r3;

        print(s1);
        print(s2);
        print(s3);
        yield(0);
    }

    on error(e) {
        yield(1);
    }
}

run Main;
```

The three `Researcher` agents run concurrently, making parallel LLM calls.

## Next Steps

Now that you've built your first agent, explore the [Language Guide](../language/syntax.md) to learn more about Sage's syntax and features.
