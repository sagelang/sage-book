# Supervision Trees

When agents crash, what happens? In v1.x Sage, the answer is simple: the error propagates to whoever spawned the agent, and it's your problem. This works for task agents — short-lived workers that do one thing and exit.

But **steward agents** — long-lived agents that maintain a domain — need something better. A `DatabaseSteward` that crashes because of a transient connection error should restart, not bring down the whole program.

Supervision trees provide **declarative crash recovery**. You declare how agents should be restarted when they fail, and the runtime handles it automatically.

## The Supervisor Declaration

A supervisor is declared with the `supervisor` keyword:

```sage
supervisor AppSupervisor {
    strategy: OneForOne

    children {
        DatabaseSteward {
            restart: Permanent
            connection_string: "postgres://localhost/myapp"
            schema_version: 0
        }

        APISteward {
            restart: Permanent
            port: 8080
        }

        MetricsCollector {
            restart: Transient
            interval_ms: 5000
        }
    }
}

run AppSupervisor;
```

When you `run` a supervisor, it spawns its children in order and monitors them. When a child exits, the supervisor applies its restart strategy.

## Restart Strategies

The `strategy` determines what happens when a child fails.

### OneForOne

Restart only the failed child. Other children continue running.

```sage
supervisor WebApp {
    strategy: OneForOne
    children {
        Worker1 { restart: Permanent }
        Worker2 { restart: Permanent }
        Worker3 { restart: Permanent }
    }
}
```

If `Worker2` crashes, only `Worker2` restarts. `Worker1` and `Worker3` are unaffected.

**Use when:** Children are independent. A database connection agent doesn't affect an API server agent.

### OneForAll

When one child fails, restart all children.

```sage
supervisor TightlyCoupled {
    strategy: OneForAll
    children {
        ConfigLoader { restart: Permanent }
        Worker1 { restart: Permanent }
        Worker2 { restart: Permanent }
    }
}
```

If any child crashes, all children are stopped and restarted together.

**Use when:** Children share state and can't function correctly if one fails. If your config loader crashes, the workers have stale config and should restart too.

### RestForOne

Restart the failed child and all children declared after it.

```sage
supervisor Pipeline {
    strategy: RestForOne
    children {
        DatabaseSteward { restart: Permanent }   // Position 1
        APISteward { restart: Permanent }        // Position 2
        FrontendSteward { restart: Permanent }   // Position 3
    }
}
```

If `APISteward` (position 2) crashes:
- `DatabaseSteward` (position 1) continues — it's before the failure
- `APISteward` (position 2) restarts — it failed
- `FrontendSteward` (position 3) restarts — it's after the failure

**Use when:** Children have dependencies in declaration order. The API steward depends on the database steward, and the frontend steward depends on the API steward. If the database fails, everything downstream should restart.

## Restart Policies

Each child has a restart policy that determines when it should be restarted.

### Permanent

Always restart, regardless of exit reason.

```sage
DatabaseSteward {
    restart: Permanent
    // ...
}
```

If the agent exits cleanly (calls `yield`), restart it. If it crashes (calls `yield` in `on error`), restart it. Permanent agents run forever — until the supervisor itself stops.

**Use for:** Core steward agents that must always be running.

### Transient

Restart only if the agent exited with an error.

```sage
MigrationRunner {
    restart: Transient
    // ...
}
```

If the agent exits cleanly, don't restart — it completed its work. If it crashes, restart it to retry.

**Use for:** Agents that do work and then should stop, but should retry on failure.

### Temporary

Never restart.

```sage
OneTimeSetup {
    restart: Temporary
    // ...
}
```

Run once. If it succeeds, fine. If it fails, fine. Don't restart either way.

**Use for:** Initialisation agents, cleanup agents, or agents that shouldn't retry.

## Restart Intensity Limiting

A crashing agent that keeps crashing creates a **restart storm**. To prevent this, supervisors have a circuit breaker:

```toml
# grove.toml
[supervision]
max_restarts = 5
within_seconds = 60
```

If a supervisor sees more than `max_restarts` within `within_seconds`, it gives up and terminates. If the supervisor has a parent supervisor, that parent's strategy applies.

Default: 5 restarts within 60 seconds.

## Integration with Persistence

Supervision and [persistent beliefs](../agents/persistence.md) work together to provide **crash recovery with state**.

When a `Permanent` agent with `@persistent` fields restarts:

1. The supervisor spawns a fresh agent instance
2. `@persistent` fields are loaded from the last checkpoint
3. `on waking` runs (validate recovered state, reconnect)
4. `on start` runs (normal operation)

```sage
agent DatabaseSteward {
    @persistent schema_version: Int
    @persistent migration_log: List<String>

    on waking {
        print("Recovered at schema v{self.schema_version.get()}");
        reconnect_to_database();
    }

    on start {
        // Resume normal operation
        yield(0);
    }
}
```

Without `@persistent`, a restarted agent starts fresh with zero-valued fields. This may be fine for stateless workers, but steward agents typically need persistence.

## Belief Initialisation

When declaring children in a supervisor, you provide initial values for their beliefs:

```sage
supervisor AppSupervisor {
    strategy: OneForOne
    children {
        QueryMonitor {
            restart: Permanent
            check_count: 0
            slow_query_threshold_ms: 100
            alert_count: 0
        }
    }
}
```

These are the **initial values** used on the first run. If the agent has `@persistent` fields and a checkpoint exists, the checkpoint values are used instead.

## Practical Example

A database guardian with multiple monitoring agents:

```sage
// Query Monitor - tracks slow queries
agent QueryMonitor {
    @persistent check_count: Int
    @persistent alert_count: Int

    on waking {
        trace("Resuming with {self.check_count.get()} previous checks");
    }

    on start {
        let count = self.check_count.get() + 1;
        self.check_count.set(count);

        trace("Check #{count}");
        // Actual monitoring logic...

        yield(count);
    }

    on error(e) {
        let alerts = self.alert_count.get() + 1;
        self.alert_count.set(alerts);
        trace("Error (alert #{alerts})");
        yield(-1);
    }
}

// Pool Monitor - watches connection pool
agent PoolMonitor {
    @persistent max_connections_seen: Int

    on start {
        let current = check_pool_connections();
        if current > self.max_connections_seen.get() {
            self.max_connections_seen.set(current);
        }
        yield(current);
    }

    on error(e) {
        yield(-1);
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
        PoolMonitor {
            restart: Permanent
            max_connections_seen: 0
        }
    }
}

run DbGuardian;
```

Configure in `grove.toml`:

```toml
[project]
name = "db-guardian"

[persistence]
backend = "sqlite"
path = ".sage/db_guardian.db"

[supervision]
max_restarts = 10
within_seconds = 30
```

## Running a Supervisor

Use `run SupervisorName;` at the end of your file:

```sage
run DbGuardian;
```

The supervisor starts all children and monitors them. The program runs until:
- All children have exited (and none need restarting)
- The circuit breaker trips (too many restarts)
- The process is killed externally

## Nested Supervisors

Supervisors can be children of other supervisors, creating a supervision tree:

```sage
supervisor DatabaseSection {
    strategy: OneForAll
    children {
        QueryMonitor { restart: Permanent }
        PoolMonitor { restart: Permanent }
    }
}

supervisor ApiSection {
    strategy: OneForOne
    children {
        RouterAgent { restart: Permanent }
        HandlerPool { restart: Permanent }
    }
}

supervisor AppRoot {
    strategy: RestForOne
    children {
        DatabaseSection { restart: Permanent }
        ApiSection { restart: Permanent }
    }
}

run AppRoot;
```

If the `DatabaseSection` supervisor's circuit breaker trips, `AppRoot` sees it as a child failure and applies `RestForOne` — restarting `DatabaseSection` and `ApiSection`.

Maximum nesting depth: 8 levels (to prevent pathological trees).

## Best Practices

1. **Start with OneForOne.** It's the simplest and usually correct. Escalate to RestForOne or OneForAll only when you have clear dependencies.

2. **Use Permanent for core stewards.** Your main agents should always be running.

3. **Use Transient for retry-on-failure workers.** Agents that do work and exit should be Transient.

4. **Pair Permanent with @persistent.** An always-restart agent without persistence restarts from scratch — probably not what you want.

5. **Tune restart intensity.** The default (5 restarts in 60 seconds) may be too aggressive or too lenient for your use case.

6. **Keep supervisors shallow.** Deep nesting is a code smell. If you need more than 2-3 levels, reconsider your architecture.

## Related

- [Persistent Beliefs](../agents/persistence.md) — State that survives restarts
- [The Steward Pattern](./pattern.md) — Building long-lived agents
- [Lifecycle Hooks](./lifecycle.md) — `on waking`, `on resting`, and friends
