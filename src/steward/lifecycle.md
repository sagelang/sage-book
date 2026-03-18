# Lifecycle Hooks

Sage agents go through a lifecycle: they wake, they start, they may pause and resume, and eventually they rest. v2.0 provides hooks for each phase, letting you run code at the right moment.

## The Full Lifecycle

```
Process start / Supervisor restart
         │
         ▼
  Load @persistent fields
         │
         ▼
  ┌─────────────┐
  │  on waking  │  ← State loaded, reconnect resources
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐
  │  on start   │  ← Main agent logic
  └──────┬──────┘
         │
    ┌────┴─────┐
    │          │
    ▼          ▼
┌────────┐  ┌───────────┐
│on pause│  │on message │  ← Concurrent with on start
└────┬───┘  └───────────┘
     │
     ▼
┌──────────┐
│on resume │
└────┬─────┘
     │
     ▼
    yield
     │
     ▼
┌────────────┐
│ on resting │  ← Cleanup before exit
└────────────┘
```

## Handler Reference

### `on waking`

Runs after `@persistent` fields are loaded from checkpoint, before `on start`.

**Use for:**
- Validating recovered state
- Reconnecting to external resources (databases, APIs)
- Registering with service registries
- Logging recovery

```sage
agent DatabaseSteward {
    @persistent schema_version: Int
    @persistent connection_string: String

    on waking {
        trace("Recovered at schema v{self.schema_version.get()}");

        if self.connection_string.get() != "" {
            reconnect_database();
            trace("Database connection re-established");
        }
    }

    on start {
        // Normal operation
        yield(0);
    }
}
```

**Restrictions:**
- Cannot call `yield` (the agent hasn't started yet)
- Cannot call `receive()` (no messages before start)

**Warning:** Using `on waking` without any `@persistent` fields is pointless — the checker emits warning W006.

### `on start`

The main entry point. Runs every time the agent starts or restarts.

**Use for:**
- Core agent logic
- Initial setup (if not recovered)
- Starting the main work loop

```sage
agent Worker {
    on start {
        trace("Worker starting");
        do_work();
        yield(0);
    }
}
```

This is the only required handler. Every agent needs `on start`.

### `on message(msg: T)`

Handles incoming messages. Can run concurrently with `on start` if the agent uses `loop` with `receive()`.

**Use for:**
- Processing commands from other agents
- Handling protocol messages
- Event-driven behaviour

```sage
agent Coordinator receives Command {
    on start {
        loop {
            let cmd: Command = receive();
            // Delegate to on message
        }
    }

    on message(cmd: Command) {
        match cmd {
            Command.Process(data) => process(data),
            Command.Shutdown => break,
        }
    }
}
```

### `on pause`

Runs when a supervisor signals a graceful pause.

**Use for:**
- Finishing in-flight work
- Flushing buffers
- Releasing locks
- Checkpointing current state

```sage
agent StreamProcessor {
    @persistent processed_count: Int
    buffer: List<Event>

    on pause {
        trace("Pausing, flushing {len(self.buffer)} buffered events");
        flush_buffer();
        trace("Pause complete");
    }
}
```

**Restrictions:**
- Cannot call `yield` (pausing is temporary)
- Should complete quickly to avoid blocking the supervisor

### `on resume`

Runs when the agent is unpaused by the supervisor.

**Use for:**
- Resuming work
- Re-acquiring resources released during pause
- Logging resume

```sage
agent StreamProcessor {
    on resume {
        trace("Resuming stream processing");
        reacquire_stream_lock();
    }
}
```

### `on resting`

Runs after `yield` is called, before the agent process exits.

**Use for:**
- Closing connections
- Flushing final state
- Deregistering from service registries
- Cleanup

```sage
agent APISteward {
    @persistent routes_generated: Bool

    on resting {
        trace("APISteward resting, cleaning up");
        close_database_connections();
        deregister_from_consul();
        trace("Cleanup complete");
    }
}
```

**Restrictions:**
- Cannot call `yield` (already yielded)
- Cannot call `receive()` (mailbox closed)

**Note:** `on resting` is the v2.0 name. `on stop` is still accepted as an alias for backward compatibility.

### `on error(e)`

Handles errors that propagate to the agent.

**Use for:**
- Logging errors
- Cleanup on failure
- Deciding whether to retry or give up

```sage
agent Worker {
    on start {
        let data = try fetch_data();  // May fail
        process(data);
        yield(0);
    }

    on error(e) {
        trace("Worker failed: {e.message}");
        // Must yield to exit
        yield(-1);
    }
}
```

**Important:** `on error` must call `yield`. If you don't, the error re-propagates.

## Lifecycle with Supervision

When a supervised agent crashes and restarts:

1. The supervisor detects the exit
2. If restart policy permits, a fresh agent is spawned
3. `@persistent` fields load from checkpoint
4. `on waking` runs
5. `on start` runs
6. Normal operation resumes

The agent picks up where it left off, thanks to persistent beliefs.

```sage
agent Counter {
    @persistent count: Int

    on waking {
        trace("Counter recovered at {self.count.get()}");
    }

    on start {
        let current = self.count.get() + 1;
        self.count.set(current);
        trace("Count is now {current}");
        yield(current);
    }
}

supervisor CounterSupervisor {
    strategy: OneForOne
    children {
        Counter {
            restart: Permanent
            count: 0
        }
    }
}
```

## Lifecycle Without Supervision

For standalone agents (no supervisor), the lifecycle is simpler:

1. Agent spawns
2. `@persistent` fields load (if any)
3. `on waking` runs (if defined)
4. `on start` runs
5. `yield` is called
6. `on resting` runs (if defined)
7. Agent exits

No restarts — a crash exits the whole program.

## Common Patterns

### First-Run vs Recovery

```sage
agent Initialiser {
    @persistent setup_complete: Bool

    on waking {
        if self.setup_complete.get() {
            trace("Recovering from previous run");
        } else {
            trace("First run, no state to recover");
        }
    }

    on start {
        if !self.setup_complete.get() {
            run_setup();
            self.setup_complete.set(true);
        }
        yield(0);
    }
}
```

### Graceful Shutdown

```sage
agent Server {
    should_shutdown: Bool

    on start {
        loop {
            if self.should_shutdown {
                break;
            }
            handle_request();
        }
        yield(0);
    }

    on message(cmd: Command) {
        if cmd == Command.Shutdown {
            trace("Shutdown requested");
            self.should_shutdown = true;
        }
    }

    on resting {
        trace("Server shutting down gracefully");
        close_all_connections();
    }
}
```

### Resource Lifecycle

```sage
agent DatabaseAgent {
    use Database
    @persistent last_query_time: String

    on waking {
        trace("Reconnecting to database");
        ensure_connection();
    }

    on start {
        loop {
            let query = receive();
            try Database.execute(query);
            self.last_query_time.set(now_iso());
        }
    }

    on resting {
        trace("Closing database connection");
        close_connection();
    }
}
```

## Handler Restrictions Summary

| Handler | Can `yield`? | Can `receive()`? | Can use tools? |
|---------|--------------|------------------|----------------|
| `on waking` | No | No | Yes |
| `on start` | Yes | Yes | Yes |
| `on message` | No (use `break`) | No (already receiving) | Yes |
| `on pause` | No | No | Yes (briefly) |
| `on resume` | No | No | Yes |
| `on resting` | No | No | Yes (briefly) |
| `on error` | Yes (required) | No | Yes |

## Related

- [Persistent Beliefs](../agents/persistence.md) — State that survives restarts
- [Supervision Trees](./supervision.md) — Restart behaviour
- [Event Handlers](../agents/handlers.md) — Handler basics
