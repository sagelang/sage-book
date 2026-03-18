# Observability

Production steward programs need visibility. When an agent crashes, you need to know what it was doing. When an `infer` call takes 8 seconds, you need to know which agent called it. When a migration fails, you need the full context.

Sage v2.0 provides structured observability as a first-class language feature.

## The `trace` Statement

Add trace events at key points in your agent logic:

```sage
agent DataProcessor {
    on start {
        trace("Starting data processing");
        let data = try load_data();
        trace("Loaded {len(data)} items");

        for item in data {
            trace("Processing: {item.id}");
            process(item);
        }

        trace("Processing complete");
        yield(len(data));
    }
}
```

Trace events include:
- Timestamp
- Agent name and ID
- Current handler
- Your message

## The `span` Block

Group related work under a named span for timing and tracing:

```sage
agent MigrationRunner {
    on start {
        span "schema reconciliation" {
            let current = get_current_version();
            let target = determine_target_version();
            apply_migrations(current, target);
        }
        // span ends here — duration is recorded automatically

        span "index rebuild" {
            rebuild_indexes();
        }

        yield(0);
    }
}
```

Nested spans create a trace tree:

```sage
span "outer" {
    trace("in outer");

    span "inner" {
        trace("in inner");
    }

    trace("back in outer");
}
```

## Configuration

### Environment Variables (Quick Start)

```bash
# Enable tracing to stderr
export SAGE_TRACE=1

# Or write to a file
export SAGE_TRACE_FILE=trace.ndjson
```

### Command Line

```bash
# Trace to stderr
sage run program.sg --trace

# Trace to file
sage run program.sg --trace-file trace.ndjson
```

### grove.toml (Recommended)

Configure the observability backend in your project manifest:

```toml
[project]
name = "my-steward"

[observability]
backend = "ndjson"    # ndjson | otlp | none
```

#### NDJSON Backend (Default)

Newline-delimited JSON output. Good for local development and log aggregation.

```toml
[observability]
backend = "ndjson"
```

Output goes to stderr by default, or to a file if `SAGE_TRACE_FILE` is set.

#### OTLP Backend

OpenTelemetry Protocol HTTP/JSON export. Integrates with Grafana, Jaeger, Honeycomb, and any OTLP-compatible backend.

```toml
[observability]
backend = "otlp"
otlp_endpoint = "http://localhost:4318/v1/traces"
service_name = "my-steward"
```

#### Disabled

Turn off tracing entirely:

```toml
[observability]
backend = "none"
```

## Automatic Events

The runtime emits automatic trace events for:

| Event | When |
|-------|------|
| `agent.spawn` | Agent spawned |
| `agent.start` | `on start` handler begins |
| `agent.emit` | Agent emits result |
| `agent.error` | `on error` handler triggered |
| `agent.stop` | `on resting` handler runs |
| `infer.start` | LLM call begins |
| `infer.complete` | LLM call completes |
| `infer.error` | LLM call fails |
| `span.start` | `span` block begins |
| `span.end` | `span` block completes |
| `user` | Custom `trace()` event |

For supervised agents, additional events:
| Event | When |
|-------|------|
| `supervisor.start` | Supervisor starts monitoring |
| `supervisor.child.restart` | Child agent restarted |
| `supervisor.circuit_breaker` | Restart limit exceeded |

## NDJSON Format

Events are emitted as newline-delimited JSON:

```json
{"t":1710000000001,"kind":"agent.spawn","agent":"Worker","id":"abc123"}
{"t":1710000000002,"kind":"agent.start","agent":"Worker","id":"abc123"}
{"t":1710000000003,"kind":"user","message":"Processing batch 1"}
{"t":1710000000015,"kind":"infer.start","agent":"Worker","id":"abc123","model":"gpt-4o","prompt_len":150}
{"t":1710000000842,"kind":"infer.complete","agent":"Worker","id":"abc123","model":"gpt-4o","response_len":320,"duration_ms":827}
{"t":1710000000843,"kind":"agent.emit","agent":"Worker","id":"abc123","value_type":"String"}
```

This format is compatible with `jq`, Elasticsearch, Datadog, and standard log aggregation tools.

## Analysing Traces

### Pretty Print

```bash
sage trace pretty trace.ndjson
```

Output:
```
[0.000s] agent.spawn    Worker
[0.001s] agent.start    Worker
[0.002s] user           "Processing batch 1"
[0.014s] infer.start    Worker        model=gpt-4o
[0.841s] infer.complete Worker        827ms
[0.842s] agent.emit     Worker
```

### Summary Statistics

```bash
sage trace summary trace.ndjson
```

Output:
```
Trace Summary
─────────────────────────────────
Duration:        1.204s
Agents spawned:  3
LLM calls:       5

Agent Timeline:
  Coordinator    0.000s - 0.904s (904ms)
  Worker         0.002s - 0.902s (900ms)

LLM Statistics:
  Total calls:   5
  Total time:    3.2s
  Avg duration:  640ms
  Success rate:  100%
```

### Filter Events

```bash
# By agent
sage trace filter trace.ndjson --agent Worker

# By event kind
sage trace filter trace.ndjson --kind infer.complete

# By time range
sage trace filter trace.ndjson --after 0.5 --before 1.0
```

### LLM Analysis

```bash
sage trace infer trace.ndjson
```

Output:
```
LLM Calls
───────────────────────────────────────────────────
Agent       Model     Duration  Status
───────────────────────────────────────────────────
Worker      gpt-4o    827ms     OK
Worker      gpt-4o    912ms     OK
───────────────────────────────────────────────────
Total: 2 calls, 1739ms, 100% success
```

## OTLP Integration

With OTLP configured, traces are exported to your OpenTelemetry collector:

```toml
[observability]
backend = "otlp"
otlp_endpoint = "http://localhost:4318/v1/traces"
service_name = "database-guardian"
```

### Grafana Tempo

```yaml
# docker-compose.yml
services:
  tempo:
    image: grafana/tempo:latest
    ports:
      - "4318:4318"  # OTLP HTTP
```

```toml
[observability]
backend = "otlp"
otlp_endpoint = "http://localhost:4318/v1/traces"
```

### Jaeger

```yaml
services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "4318:4318"  # OTLP HTTP
      - "16686:16686" # UI
```

### Honeycomb

```toml
[observability]
backend = "otlp"
otlp_endpoint = "https://api.honeycomb.io/v1/traces"
service_name = "my-steward"
```

Set `HONEYCOMB_API_KEY` environment variable.

## Best Practices

### 1. Trace at Boundaries

Add traces at the start and end of significant operations:

```sage
trace("Starting batch processing");
// ... work ...
trace("Batch complete: {count} items processed");
```

### 2. Use Spans for Timing

Wrap timed operations in spans:

```sage
span "database migration" {
    apply_migration(migration);
}
// Duration automatically recorded
```

### 3. Include Context

Add relevant data to trace messages:

```sage
trace("Processing user {user.id}: {user.email}");
trace("Query returned {len(rows)} rows");
```

### 4. Monitor in Production

Use OTLP export for production observability:

```toml
[observability]
backend = "otlp"
otlp_endpoint = "https://your-collector.example.com/v1/traces"
service_name = "production-steward"
```

### 5. Analyse LLM Costs

Use trace analysis to understand LLM usage:

```bash
sage trace infer production-trace.ndjson
# Identify slow calls, high token counts, failure patterns
```

## Related

- [Error Handling](../language/error-handling.md) — Error events in traces
- [Supervision Trees](../steward/supervision.md) — Supervisor events
- [LLM Configuration](../llm/configuration.md) — Model settings affecting traces
