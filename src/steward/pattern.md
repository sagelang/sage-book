# The Steward Pattern

Sage v1.x proved the thesis: agents, beliefs, and LLM inference as first-class language constructs produce programs that are simpler and safer than equivalent Python framework code.

Sage v2.0 pursues a deeper thesis: **agents as stewards of long-lived systems**.

## What Is a Steward?

A **steward** is an agent that:
- Owns a domain
- Maintains it over time
- Reacts to change
- Coordinates with other stewards
- Survives crashes

The most valuable systems in software are not tasks — they are ongoing processes. A database doesn't run once and exit. An API server doesn't emit a result and terminate. These are stewards.

## The Steward Anatomy

A complete steward agent has:

```sage
agent DatabaseSteward
    uses Database                              // Tool capabilities
    follows SchemaSync as DatabaseSteward      // Protocol participation
    receives SchemaCommand {                   // Message type

    @persistent schema_version: Int            // Durable state
    @persistent migration_log: List<String>

    active_connections: Int                    // Ephemeral state

    on waking {                                // Recovery hook
        trace("Recovered at schema v{self.schema_version.get()}");
        reconnect_database();
    }

    on start {                                 // Main logic
        loop {
            let cmd: SchemaCommand = receive();
            handle_command(cmd);
        }
        yield(0);
    }

    on message(cmd: SchemaCommand) {           // Message handler
        // Protocol-aware handling
        match cmd {
            SchemaCommand.Migrate(spec) => {
                apply_migration(spec);
                reply(Acknowledged {});
            }
        }
    }

    on resting {                               // Cleanup hook
        trace("Shutting down");
        close_connections();
    }

    on error(e) {                              // Error handler
        trace("Error: {e.message}");
        yield(1);
    }
}
```

## The Three-Steward Architecture

The canonical steward application is a web application expressed as three coordinating stewards:

```
┌─────────────────────────────────────────────────┐
│                  AppSupervisor                  │
│           strategy: RestForOne                  │
│                                                 │
│  ┌───────────────────────────────────────────┐  │
│  │ DatabaseSteward    restart: Permanent     │  │
│  │   @persistent schema_version              │  │
│  │   uses: Database                          │  │
│  │   follows: SchemaSync as DatabaseSteward  │  │
│  └───────────────────┬───────────────────────┘  │
│                      │ SchemaChanged            │
│                      ▼                          │
│  ┌───────────────────────────────────────────┐  │
│  │ APISteward        restart: Permanent      │  │
│  │   @persistent route_version               │  │
│  │   uses: Database, Http, Fs                │  │
│  │   follows: SchemaSync as APISteward       │  │
│  │   follows: ApiSync as APISteward          │  │
│  └───────────────────┬───────────────────────┘  │
│                      │ RouteChanged             │
│                      ▼                          │
│  ┌───────────────────────────────────────────┐  │
│  │ FrontendSteward   restart: Permanent      │  │
│  │   @persistent build_version               │  │
│  │   uses: Fs, Shell                         │  │
│  │   follows: ApiSync as FrontendSteward     │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

### Why RestForOne?

The `RestForOne` strategy is deliberate:
- If `DatabaseSteward` crashes, both `APISteward` and `FrontendSteward` restart (they depend on the database)
- If `APISteward` crashes, only `FrontendSteward` restarts (it depends on the API)
- If `FrontendSteward` crashes, only it restarts (nothing depends on it)

The dependencies flow downward in declaration order.

## Change Propagation

The key pattern is **declarative change propagation**:

1. `DatabaseSteward` detects a schema change (via LLM reasoning or external command)
2. `DatabaseSteward` applies the migration via `Database.execute`
3. `DatabaseSteward` increments `schema_version` (checkpointed)
4. `DatabaseSteward` sends `SchemaChanged(change)` to `APISteward`
5. `APISteward` receives, regenerates affected routes via `infer`
6. `APISteward` increments `route_version` (checkpointed)
7. `APISteward` sends `RouteChanged(change)` to `FrontendSteward`
8. `FrontendSteward` regenerates affected components
9. `FrontendSteward` runs build, updates `build_version` (checkpointed)

At any point, a crash is safe: the checkpointed state tells the restarted steward exactly where it left off.

## Why This Matters

### Versus Frameworks

Every framework that attempts this today (LangChain, AutoGen, CrewAI) hits the same ceiling: coordination logic is in Python, so the framework can't reason about it.

| Problem | Framework Approach | Sage Approach |
|---------|-------------------|---------------|
| Agent crashes | Restart blindly | Typed supervision with state recovery |
| Inter-agent communication | String messages | Session-typed protocols |
| Tool failures | Python exceptions | Typed `Result<T, ToolError>` |
| State persistence | Manual serialisation | `@persistent` annotation |

### Versus Manual Code

You could write this in Rust or Python without Sage. But you'd be implementing:
- Your own checkpoint system
- Your own supervision tree
- Your own protocol verification
- Your own LLM integration
- Your own observability

Sage provides these as language features. The **compiler** is the primary safety mechanism, not the programmer's vigilance.

## Building a Steward

### Step 1: Define Your Domain

What does this steward own? What state must survive restarts?

```sage
agent CacheManager {
    @persistent cache_version: Int
    @persistent eviction_count: Int
    @persistent last_cleanup: String
}
```

### Step 2: Declare Capabilities

What external resources does this steward need?

```sage
agent CacheManager uses Database, Http {
    // Can access both Database and Http tools
}
```

### Step 3: Define Protocols

How does this steward communicate with others?

```sage
protocol CacheInvalidation {
    DataSteward -> CacheManager: InvalidateKey
    CacheManager -> DataSteward: Acknowledged
}

agent CacheManager follows CacheInvalidation as CacheManager {
    // Protocol-compliant communication
}
```

### Step 4: Implement Handlers

Write the lifecycle hooks:

```sage
agent CacheManager {
    on waking {
        trace("Cache restored at version {self.cache_version.get()}");
        warm_cache();
    }

    on start {
        loop {
            let cmd = receive();
            process(cmd);
        }
    }

    on resting {
        trace("Flushing cache to disk");
        flush_to_disk();
    }
}
```

### Step 5: Wrap in Supervisor

Declare the restart behaviour:

```sage
supervisor CacheSystem {
    strategy: OneForOne

    children {
        CacheManager {
            restart: Permanent
            cache_version: 0
            eviction_count: 0
            last_cleanup: ""
        }
    }
}
```

### Step 6: Configure

Set up persistence and observability:

```toml
# grove.toml
[project]
name = "cache-system"

[persistence]
backend = "sqlite"
path = ".sage/cache_state.db"

[supervision]
max_restarts = 5
within_seconds = 60

[observability]
backend = "otlp"
otlp_endpoint = "http://localhost:4318/v1/traces"
```

## Example: Database Guardian

A complete steward application that monitors a PostgreSQL database:

```sage
// Types
record QueryStats {
    slow_count: Int,
    avg_ms: Float,
}

// Protocols
protocol AlertProtocol {
    QueryMonitor -> AlertSender: Alert
    AlertSender -> QueryMonitor: Acknowledged
}

record Alert {
    severity: String,
    message: String,
}

record Acknowledged {}

// Query Monitor Steward
agent QueryMonitor
    uses Database
    follows AlertProtocol as QueryMonitor {

    @persistent check_count: Int
    @persistent alert_count: Int

    on waking {
        trace("Resuming monitoring, {self.check_count.get()} checks done");
    }

    on start {
        loop {
            span "monitoring cycle" {
                let stats = try Database.query(
                    "SELECT COUNT(*) as cnt, AVG(mean_exec_time) as avg " ++
                    "FROM pg_stat_statements WHERE mean_exec_time > 500"
                );

                self.check_count.set(self.check_count.get() + 1);

                if has_problems(stats) {
                    send(alert_sender, Alert {
                        severity: "warning",
                        message: "Slow queries detected",
                    });
                    self.alert_count.set(self.alert_count.get() + 1);
                }
            }

            sleep_ms(60000);  // Check every minute
        }
    }

    on error(e) {
        trace("Monitor error: {e.message}");
        yield(1);
    }
}

// Alert Sender Steward
agent AlertSender
    uses Http
    follows AlertProtocol as AlertSender
    receives Alert {

    @persistent alerts_sent: Int

    on message(alert: Alert) {
        let payload = json_stringify(alert);
        try Http.post("https://hooks.slack.com/...", payload);
        self.alerts_sent.set(self.alerts_sent.get() + 1);
        reply(Acknowledged {});
    }

    on start {
        yield(0);
    }
}

// Supervisor
supervisor DbGuardian {
    strategy: OneForOne

    children {
        QueryMonitor {
            restart: Permanent
            check_count: 0
            alert_count: 0
        }
        AlertSender {
            restart: Permanent
            alerts_sent: 0
        }
    }
}

run DbGuardian;
```

## When to Use Stewards

Use the steward pattern when:

- **Continuous operation**: The system should run indefinitely
- **State matters**: Losing state on crash would be costly
- **Coordination required**: Multiple agents must work together
- **Recovery is important**: Crashes should be handled gracefully

Don't use stewards for:

- **One-shot tasks**: Simple agents that run once and exit
- **Stateless workers**: Agents that don't need to remember anything
- **Single-agent programs**: No coordination needed

## Summary

The steward pattern combines:

| Feature | Purpose |
|---------|---------|
| `@persistent` beliefs | State that survives restarts |
| `supervisor` declarations | Automatic crash recovery |
| `uses` clauses | Typed tool capabilities |
| `follows` clauses | Protocol-verified communication |
| Lifecycle hooks | Resource management |
| Observability | Production visibility |

Together, these features enable **infrastructure-as-intent**: you declare what you want each steward to maintain, and the language runtime ensures it is maintained.

## Related

- [Persistent Beliefs](../agents/persistence.md) — State management
- [Supervision Trees](./supervision.md) — Crash recovery
- [Session Types](./sessions.md) — Protocol verification
- [Lifecycle Hooks](./lifecycle.md) — Resource management
- [Tools](../tools/overview.md) — External capabilities
- [Observability](../observability/overview.md) — Production visibility
