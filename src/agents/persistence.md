# Persistent Beliefs

Sage agents are, by default, ephemeral. When an agent completes or crashes, its state is gone. For task agents this is fine — they do their work and vanish. But **steward agents** — long-lived agents that maintain a domain over time — need to survive restarts.

Persistent beliefs solve this. Mark a field with `@persistent` and Sage will checkpoint it to durable storage. When the agent restarts, its state is recovered automatically.

## Basic Usage

```sage
agent Counter {
    @persistent count: Int

    on start {
        let current = self.count.get();
        print("Starting at count: {current}");

        self.count.set(current + 1);
        yield(current);
    }
}

run Counter;
```

Run this program multiple times. You'll see the count increment across restarts:

```
$ sage run counter.sg
Starting at count: 0

$ sage run counter.sg
Starting at count: 1

$ sage run counter.sg
Starting at count: 2
```

## The `@persistent` Annotation

Add `@persistent` before any agent field to enable checkpointing:

```sage
agent DatabaseSteward {
    @persistent schema_version: Int
    @persistent migration_log: List<String>
    @persistent last_sync: String

    // Non-persistent — recomputed on every start
    active_connections: Int

    on start {
        // schema_version, migration_log, and last_sync are already
        // populated from the last checkpoint (or zero-valued on first run)
        print("Schema at version {self.schema_version.get()}");
        yield(0);
    }
}
```

### Accessing Persistent Fields

Persistent fields use a wrapper that provides `.get()` and `.set()` methods:

```sage
// Read the current value
let version = self.schema_version.get();

// Update and checkpoint atomically
self.schema_version.set(version + 1);
```

Every `.set()` call immediately checkpoints the value. A crash after `.set()` will not lose that update.

### Serialisable Types

Only serialisable types can be `@persistent`. These are:

- Primitives: `Int`, `Float`, `Bool`, `String`
- Collections: `List<T>`, `Map<K, V>` (where `T`, `K`, `V` are serialisable)
- `Option<T>` and `Result<T, E>` (where inner types are serialisable)
- Records (where all fields are serialisable)
- Enums (including payload-carrying variants)

Function types and agent handles cannot be persisted — this is a compile error:

```sage
agent Invalid {
    @persistent callback: Fn(Int) -> Int  // Error E052: not serialisable
}
```

## First-Run Detection

A common pattern is detecting whether an agent is starting fresh or recovering from a checkpoint:

```sage
agent APISteward {
    @persistent initialised: Bool

    on start {
        if !self.initialised.get() {
            // First run — do expensive setup
            print("First run: generating routes...");
            generate_routes();
            self.initialised.set(true);
        } else {
            // Subsequent run — state already loaded
            print("Recovered from checkpoint");
        }

        yield(0);
    }
}
```

For more complex cases, check if specific fields have meaningful values:

```sage
agent ConfigManager {
    @persistent config_hash: String

    on start {
        if self.config_hash.get() == "" {
            // No config loaded yet
            let config = load_config_file();
            self.config_hash.set(hash(config));
        }
        yield(0);
    }
}
```

## The `on waking` Lifecycle Hook

When an agent with persistent fields restarts, you often need to validate or act on the recovered state before normal operation begins. The `on waking` hook runs after persistent state is loaded but before `on start`:

```sage
agent DatabaseSteward {
    @persistent schema_version: Int
    @persistent connection_string: String

    on waking {
        // State is already loaded — validate it
        print("Recovered at schema version {self.schema_version.get()}");

        // Reconnect to resources
        if self.connection_string.get() != "" {
            reconnect_database();
        }
    }

    on start {
        // Normal operation begins
        yield(0);
    }
}
```

The lifecycle sequence is:

```
Process start / Restart
        │
        ▼
  Load checkpoint
        │
        ▼
  ┌─────────────┐
  │  on waking  │  ← Persistent state available, validate/reconnect
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐
  │  on start   │  ← Normal agent logic
  └──────┬──────┘
         │
         ▼
    ... run ...
         │
         ▼
  ┌─────────────┐
  │ on resting  │  ← Cleanup before exit
  └─────────────┘
```

## Storage Backends

Configure the persistence backend in `grove.toml`:

### SQLite (Default)

Best for local development and single-machine deployments:

```toml
[persistence]
backend = "sqlite"
path = ".sage/checkpoints.db"
```

### PostgreSQL

Recommended for production steward programs:

```toml
[persistence]
backend = "postgres"
url = "postgresql://user:pass@localhost/myapp"
```

### File

JSON files, useful for debugging:

```toml
[persistence]
backend = "file"
path = ".sage/state"
```

Each agent gets a separate JSON file in the directory.

## Checkpoint Namespacing

Each agent instance has a unique checkpoint namespace derived from:
1. The agent name
2. Its initial belief values

This means two agents of the same type with different initial beliefs have independent checkpoints:

```sage
supervisor TwoCounters {
    strategy: OneForOne
    children {
        Counter { restart: Permanent, count: 0 }   // Checkpoint key: Counter_abc123
        Counter { restart: Permanent, count: 100 } // Checkpoint key: Counter_def456
    }
}
```

## Integration with Supervision

Persistent beliefs and supervision work together to provide crash recovery with state:

```sage
supervisor AppSupervisor {
    strategy: OneForOne
    children {
        DatabaseSteward {
            restart: Permanent
            schema_version: 0
            migration_log: []
        }
    }
}
```

When a `Permanent` agent crashes and restarts:
1. The supervisor respawns the agent
2. Persistent fields are loaded from the last checkpoint
3. `on waking` runs with recovered state
4. `on start` runs as normal

The agent resumes from its last stable checkpoint, not from scratch.

## Explicit Checkpointing

Normally, `.set()` checkpoints automatically. For batched updates, you can checkpoint explicitly:

```sage
agent BatchUpdater {
    @persistent items: List<String>

    on start {
        // Make many updates without individual checkpoints
        let mut current = self.items.get();
        for i in range(0, 100) {
            current = push(current, "item_{i}");
        }

        // Checkpoint once at the end
        self.items.set(current);

        yield(0);
    }
}
```

## Error Handling

If a checkpoint fails (database unavailable, disk full, etc.), the agent continues running but logs a warning. The next successful checkpoint will include the latest state.

For critical applications, you can catch persistence errors in your `on error` handler — though typically the supervision tree handles this by restarting the agent.

## Best Practices

1. **Checkpoint only what you need.** Every `.set()` is a write operation. Don't persist fields that can be recomputed cheaply.

2. **Keep persistent fields small.** Large lists or maps checkpoint slowly. Consider aggregating or summarising data.

3. **Use `on waking` for validation.** If your agent depends on external resources (database connections, file handles), re-establish them in `on waking`.

4. **Test recovery.** Write tests that simulate crashes and verify your agent recovers correctly.

5. **Consider checkpoint frequency.** For high-frequency updates, batch changes and checkpoint periodically rather than on every update.

## Related

- [Supervision Trees](../steward/supervision.md) — Automatic restart on failure
- [Lifecycle Hooks](./handlers.md) — All agent lifecycle events
- [The Steward Pattern](../steward/pattern.md) — Building long-lived agents
