# Messaging

Agents can receive typed messages from other agents using the actor model pattern.

## The `receives` Clause

An agent declares what type of messages it accepts using the `receives` clause:

```sage
enum WorkerMsg {
    Task,
    Ping,
    Shutdown,
}

agent Worker receives WorkerMsg {
    id: Int

    on start {
        // This agent can now receive WorkerMsg messages
        emit(0);
    }
}
```

Agents without a `receives` clause are pure spawn/await agents and cannot receive messages.

## The `receive()` Expression

Inside an agent with a `receives` clause, use `receive()` to wait for a message:

```sage
agent Worker receives WorkerMsg {
    id: Int

    on start {
        let msg: WorkerMsg = receive();
        match msg {
            Task => print("Got a task"),
            Ping => print("Pinged"),
            Shutdown => print("Shutting down"),
        }
        emit(0);
    }
}
```

`receive()` blocks until a message arrives in the agent's mailbox.

## The `send()` Function

Send a message to a running agent using its handle. `send` is fallible (the agent might have terminated), so use `try`:

```sage
agent Main {
    on start {
        let w = spawn Worker { id: 1 };
        try send(w, Task);
        try send(w, Shutdown);
        try await w;
        emit(0);
    }

    on error(e) {
        emit(1);
    }
}

run Main;
```

`send` queues the message and returns immediately.

## Long-Running Agents with `loop`

Combine `receive()` with `loop` for agents that process multiple messages:

```sage
agent Worker receives WorkerMsg {
    id: Int

    on start {
        loop {
            let msg: WorkerMsg = receive();
            match msg {
                Task => {
                    let result = try infer("Process a task");
                    print("Worker {self.id}: {result}");
                }
                Ping => {
                    print("Worker {self.id} is alive");
                }
                Shutdown => {
                    break;
                }
            }
        }
        emit(0);
    }

    on error(e) {
        print("Worker {self.id} failed: " ++ e);
        emit(1);
    }
}
```

## Complete Example: Worker Pool

```sage
enum WorkerMsg {
    Task,
    Shutdown,
}

agent Worker receives WorkerMsg {
    id: Int

    on start {
        loop {
            let msg: WorkerMsg = receive();
            match msg {
                Task => {
                    let result = try infer("Summarise something interesting");
                    print("Worker {self.id}: {result}");
                }
                Shutdown => {
                    break;
                }
            }
        }
        emit(0);
    }

    on error(e) {
        print("Worker {self.id} failed");
        emit(1);
    }
}

agent Coordinator {
    on start {
        let w1 = spawn Worker { id: 1 };
        let w2 = spawn Worker { id: 2 };

        // Distribute tasks
        try send(w1, Task);
        try send(w2, Task);
        try send(w1, Task);
        try send(w2, Task);

        // Shut down workers
        try send(w1, Shutdown);
        try send(w2, Shutdown);

        // Wait for completion
        try await w1;
        try await w2;

        emit(0);
    }

    on error(e) {
        print("Coordination failed");
        emit(1);
    }
}

run Coordinator;
```

## Type Safety

The compiler ensures type safety:

```sage
agent Worker receives WorkerMsg {
    on start {
        let msg: WorkerMsg = receive();
        emit(0);
    }
}

agent Main {
    on start {
        let w = spawn Worker {};
        try send(w, Task);       // OK - Task is a WorkerMsg variant
        try send(w, "hello");    // Error: expected WorkerMsg, got String
        emit(0);
    }

    on error(e) {
        emit(1);
    }
}
```

## Messaging vs Awaiting

| | `await` | `send` / `receive` |
|---|---------|---------------------|
| Direction | Get final result from agent | Ongoing communication |
| Blocking | Yes, waits for agent to complete | `send` returns immediately, `receive` blocks until message arrives |
| Use case | One-shot tasks | Long-running workers, event loops |

## Mailbox Semantics

- Each agent has a **bounded mailbox** (128 messages by default)
- When the mailbox is full, `send` blocks until space opens (backpressure)
- Messages from a single sender arrive in order
- Messages from multiple senders are interleaved (no global ordering)

## Current Limitations

- No `receive_timeout` in the language yet (available in runtime)
- No broadcast channels (one-to-many messaging)
- Error handling for closed channels needs its own RFC
