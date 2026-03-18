# Session Types

When agents communicate, they follow protocols. A coordinator sends a task to a worker, the worker sends a result back. A database steward notifies an API steward of a schema change, the API steward acknowledges.

In v1.x Sage, these protocols exist only in the programmer's head. You can send any message at any time. Send a shutdown before a task. Send a result after the protocol has ended. The compiler doesn't know and doesn't care.

**Session types** make communication protocols explicit. You declare the protocol, and the compiler verifies that agents follow it. Wrong message order? Compile error. Missing reply? Compile error.

## Protocol Declarations

A protocol defines the valid sequence of messages between roles:

```sage
protocol SchemaSync {
    DatabaseSteward -> APISteward: SchemaChanged
    APISteward -> DatabaseSteward: Acknowledged
}
```

This declares:
1. `DatabaseSteward` sends a `SchemaChanged` message to `APISteward`
2. `APISteward` replies with an `Acknowledged` message

The protocol has two **roles** (`DatabaseSteward`, `APISteward`) and two **steps**.

### Multi-Step Protocols

Protocols can have multiple steps:

```sage
protocol DebateRound {
    Coordinator -> Debater: Topic
    Debater -> Coordinator: Argument
    Coordinator -> Debater: Feedback
    Debater -> Coordinator: Revision
}
```

### Message Types

Protocol steps reference message types. Define these as records or enums:

```sage
record SchemaChanged {
    table: String,
    change_type: String,
}

record Acknowledged {}

protocol SchemaSync {
    DatabaseSteward -> APISteward: SchemaChanged
    APISteward -> DatabaseSteward: Acknowledged
}
```

## Following Protocols

Agents declare which protocols they participate in using the `follows` clause:

```sage
agent APISteward
    receives SchemaChanged
    follows SchemaSync as APISteward {

    on start {
        // Wait for schema changes
        yield(0);
    }

    on message(change: SchemaChanged) {
        print("Schema changed: {change.table}");

        // Protocol requires a reply
        reply(Acknowledged {});
    }
}
```

The `follows SchemaSync as APISteward` declaration says: "This agent plays the `APISteward` role in the `SchemaSync` protocol."

## The `reply` Expression

When a protocol step expects a reply, use the `reply()` expression:

```sage
on message(change: SchemaChanged) {
    // Handle the change...
    process_schema_change(change);

    // Send the required reply
    reply(Acknowledged {});
}
```

`reply()` sends a message back to the sender of the most recent message. It's only valid inside `on message` handlers when the agent follows a protocol that expects a reply.

### Compile-Time Verification

If you forget the reply:

```sage
agent APISteward follows SchemaSync as APISteward {
    on message(change: SchemaChanged) {
        process_schema_change(change);
        // Missing reply(Acknowledged {})!
    }
}
```

The compiler catches this:

```
Error E076: Protocol SchemaSync requires APISteward to send Acknowledged after receiving SchemaChanged
```

## Protocol Errors

The checker catches protocol violations at compile time:

### E070: Unknown Protocol

```sage
agent Worker follows NonexistentProtocol as Worker {
    // Error E070: Unknown protocol 'NonexistentProtocol'
}
```

### E071: Unknown Role

```sage
protocol SchemaSync {
    DatabaseSteward -> APISteward: SchemaChanged
}

agent Worker follows SchemaSync as UnknownRole {
    // Error E071: Role 'UnknownRole' not found in protocol 'SchemaSync'
}
```

### E073: Reply Outside Handler

```sage
agent Worker follows SchemaSync as APISteward {
    on start {
        reply(Acknowledged {});  // Error E073: reply outside message handler
    }
}
```

### E074: Wrong Message Type

```sage
agent APISteward follows SchemaSync as APISteward {
    on message(change: SchemaChanged) {
        reply(WrongType {});  // Error E074: Protocol expects Acknowledged, got WrongType
    }
}
```

### E076: Missing Reply

```sage
agent APISteward follows SchemaSync as APISteward {
    on message(change: SchemaChanged) {
        print("Got change");
        // Error E076: Missing required reply
    }
}
```

## Practical Example

A request-response protocol between a client and server:

```sage
// Message types
record Request {
    data: String,
}

record Response {
    result: String,
}

// Protocol declaration
protocol RequestResponse {
    Client -> Server: Request
    Server -> Client: Response
}

// Server agent
agent RequestWorker
    receives Request
    follows RequestResponse as Server {

    on start {
        yield(0);
    }

    on message(req: Request) {
        let result = process(req.data);
        reply(Response { result: result });
    }
}

// Client agent
agent Requester
    follows RequestResponse as Client {

    target: Agent<Int>,

    on start {
        send(self.target, Request { data: "hello" });

        // Wait for response
        let response: Response = receive();
        print("Got: {response.result}");

        yield(0);
    }
}
```

## Multiple Protocols

An agent can follow multiple protocols:

```sage
agent APISteward
    follows SchemaSync as APISteward
    follows ApiSync as APISteward {

    // This agent participates in both protocols
}
```

Each `follows` clause is independent. The agent must satisfy all protocol obligations.

## Protocol State and Supervision

When a supervised agent crashes and restarts, its protocol state is reset. The restarted agent begins fresh from the protocol's initial state.

For protocols that span multiple message exchanges, consider:

1. **Idempotent operations**: Design handlers so replaying a message is safe
2. **State in persistent beliefs**: Store protocol progress in `@persistent` fields
3. **Acknowledgment patterns**: Use explicit acknowledgments to confirm each step

## Design Guidelines

### Keep Protocols Simple

Protocols with many steps are hard to reason about. Prefer short, focused protocols:

```sage
// Good: Simple two-step protocol
protocol SchemaSync {
    DatabaseSteward -> APISteward: SchemaChanged
    APISteward -> DatabaseSteward: Acknowledged
}

// Avoid: Complex multi-step protocol
protocol ComplexWorkflow {
    A -> B: Step1
    B -> A: Step2
    A -> C: Step3
    C -> A: Step4
    A -> B: Step5
    B -> C: Step6
    // ...getting hard to follow
}
```

Break complex workflows into multiple simpler protocols.

### Use Descriptive Role Names

Protocol roles should match the agent names that play them:

```sage
// Good: Role names match agent names
protocol SchemaSync {
    DatabaseSteward -> APISteward: SchemaChanged
}

agent DatabaseSteward follows SchemaSync as DatabaseSteward { }
agent APISteward follows SchemaSync as APISteward { }
```

### Document Protocol Semantics

The type system checks syntax, not semantics. Document what each message means:

```sage
// SchemaChanged: Sent when the database schema has been modified.
// The receiver should regenerate any cached schema information.
record SchemaChanged {
    table: String,
    change_type: String,  // "add_column" | "drop_column" | "add_table"
}
```

## Limitations

Session types in Sage v2.0 are **structural**, not **behavioural**. The compiler verifies:

- Protocols exist
- Roles exist in protocols
- Message types match protocol steps
- Required replies are present

The compiler does **not** verify:

- Messages are sent in the correct runtime order
- Protocol conversations terminate correctly
- State machines are followed exactly

Full behavioural session type verification is planned for v3.0.

## Related

- [Messaging](../agents/messaging.md) — Basic agent communication
- [Supervision Trees](./supervision.md) — Restart behaviour affects protocol state
- [The Steward Pattern](./pattern.md) — Protocols in steward architectures
