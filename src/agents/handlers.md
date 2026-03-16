# Event Handlers

Agents respond to events through handlers. Each handler runs when its corresponding event occurs.

## on start

Runs when the agent is summoned:

```sage
agent Worker {
    on start {
        print("Worker started!");
        yield(42);
    }
}
```

Every agent must have an `on start` handler — it's where the agent's main logic lives.

## on error

Handles errors propagated by `try`:

```sage
agent Researcher {
    topic: String

    on start {
        let result = try divine("Summarize: {self.topic}");
        yield(result);
    }

    on error(e) {
        print("Research failed: " ++ e);
        yield("unavailable");
    }
}
```

When a `try` expression fails, control jumps to `on error`. Without an `on error` handler, the agent will panic.

## Message Handling

For agents that receive messages, use the `receives` clause with `receive()`:

```sage
enum Command {
    Ping,
    Shutdown,
}

agent Worker receives Command {
    on start {
        loop {
            let msg: Command = receive();
            match msg {
                Ping => print("Pong!"),
                Shutdown => break,
            }
        }
        yield(0);
    }
}
```

See [Messaging](./messaging.md) for details.

## Handler Order

1. `on start` runs first, exactly once
2. `on error` runs if a `try` expression fails
3. After `emit`, the agent terminates

## emit

The `emit` expression signals that the agent has produced its result:

```sage
agent Calculator {
    a: Int
    b: Int

    on start {
        let result = self.a + self.b;
        yield(result);  // Agent is done
    }
}
```

After `emit`:
- The agent's result is available to whoever awaited it
- The agent proceeds to cleanup (`on stop`)
- No more messages are processed

## Emit Type Consistency

All `emit` calls in an agent must have the same type:

```sage
agent Example {
    on start {
        if condition {
            yield(42);      // Int
        } else {
            yield("error"); // Error: expected Int, got String
        }
    }
}
```

## Handler Scope

Each handler has its own scope. Variables don't persist between handlers:

```sage
agent Example {
    on start {
        let x = 42;
        // x is only visible here
        yield(0);
    }

    on error(e) {
        // x is not visible here
        // Use agent fields for persistent state
        yield(1);
    }
}
```

Use agent fields (accessed via `self`) for state that needs to persist.
