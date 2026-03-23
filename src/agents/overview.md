# What Are Agents?

Agents are the core abstraction in Sage — autonomous units of computation with state and behavior.

## The Mental Model

Think of an agent as a small, focused worker:
- It has **state** (its private fields)
- It responds to **events** (start, messages, errors)
- It can **summon** other agents
- It **yields** a result when done

```sage
agent Worker {
    task: String              // State

    on start {                // Event handler
        let result = do_work(self.task);
        yield(result);         // Result
    }
}
```

## Why Agents?

### vs. Functions
Functions are synchronous and stateless. Agents are asynchronous and maintain state across their lifetime.

### vs. Objects
Objects bundle state and methods. Agents bundle state and *event handlers* — they react to events rather than being called directly.

### vs. Threads
Threads are low-level and share memory. Agents are high-level and communicate through messages. No locks, no races.

## Agent Lifecycle

1. **Summon** — Agent is created with initial state
2. **Start** — The `on start` handler runs
3. **Running** — Agent can receive messages, summon other agents
4. **Yield** — Agent produces its result
5. **Done** — Agent terminates

```
summon Worker { task: "..." }
        │
        ▼
    ┌───────┐
    │ start │ ─── on start { ... }
    └───┬───┘
        │
        ▼
    ┌────────┐
    │running │ ─── on message { ... }
    └───┬────┘
        │
        ▼
    ┌──────┐
    │yield │ ─── yield(value)
    └──────┘
```

## A Complete Example

```sage
agent Counter {
    initial: Int

    on start {
        let count = self.initial;
        let i = 0;
        while i < 5 {
            count = count + 1;
            i = i + 1;
        }
        yield(count);
    }
}

agent Main {
    on start {
        let c1 = summon Counter { initial: 0 };
        let c2 = summon Counter { initial: 100 };

        let r1 = try await c1;  // 5
        let r2 = try await c2;  // 105

        print("Results: " ++ str(r1) ++ ", " ++ str(r2));
        yield(0);
    }

    on error(e) {
        print("A counter failed");
        yield(1);
    }
}

run Main;
```

Both counters run concurrently. The main agent waits for both results.

## Next

- [State](./beliefs.md) — Agent fields
- [Event Handlers](./handlers.md) — Responding to events
- [Summoning & Awaiting](./spawning.md) — Creating and coordinating agents
- [Messaging](./messaging.md) — Communication between agents
