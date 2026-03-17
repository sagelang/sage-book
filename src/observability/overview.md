# Observability

Sage includes built-in tracing to help you understand what your agents are doing, debug issues, and optimise performance.

## Enabling Tracing

### Console Output

Enable tracing to stderr with the `--trace` flag:

```bash
sage run program.sg --trace
```

Trace events are printed as they occur, showing agent lifecycle, LLM calls, and custom trace events.

### File Output

Write traces to a file in NDJSON format:

```bash
sage run program.sg --trace-file trace.ndjson
```

This creates a newline-delimited JSON file that can be analysed with the `sage trace` commands.

## Trace Events

Sage emits the following event types:

| Event | Description |
|-------|-------------|
| `agent.spawn` | Agent was spawned |
| `agent.start` | Agent's `on start` handler began |
| `agent.emit` | Agent emitted a result |
| `agent.error` | Agent's `on error` handler was triggered |
| `agent.stop` | Agent's `on stop` handler ran |
| `infer.start` | LLM call began |
| `infer.complete` | LLM call completed (success or failure) |
| `message.send` | Message sent to an agent |
| `message.receive` | Agent received a message |
| `user` | Custom trace event from `trace()` |

## Custom Trace Events

Use the `trace()` expression to emit custom events:

```sage
agent Processor {
    items: List<String>

    on start {
        for item in self.items {
            trace("Processing item: {item}");
            // ... process item ...
        }
        trace("Completed all items");
        yield(len(self.items));
    }
}
```

Custom events appear in the trace with `kind: "user"` and include:
- Timestamp
- Agent name
- Handler name
- Your message

## Analysing Traces

### Pretty Print

View a trace file in human-readable format:

```bash
sage trace pretty trace.ndjson
```

Output:
```
[0.000s] agent.spawn    Coordinator
[0.001s] agent.start    Coordinator
[0.002s] agent.spawn    Worker (id=1)
[0.002s] agent.spawn    Worker (id=2)
[0.003s] agent.start    Worker
[0.003s] agent.start    Worker
[0.015s] infer.start    Worker        "Summarise: quantum computing"
[0.016s] infer.start    Worker        "Summarise: gene editing"
[0.842s] infer.complete Worker        200 OK (826ms)
[0.901s] infer.complete Worker        200 OK (885ms)
[0.902s] agent.emit     Worker        "Quantum computing is..."
[0.903s] agent.emit     Worker        "Gene editing refers to..."
[0.904s] agent.emit     Coordinator   0
```

### Summary

Get high-level statistics:

```bash
sage trace summary trace.ndjson
```

Output:
```
Trace Summary
─────────────────────────────────
Duration:        1.204s
Agents spawned:  3
LLM calls:       2

Agent Timeline:
  Coordinator    0.000s - 0.904s (904ms)
  Worker         0.002s - 0.902s (900ms)
  Worker         0.002s - 0.903s (901ms)

LLM Statistics:
  Total calls:   2
  Total time:    1.711s
  Avg duration:  855ms
  Success rate:  100%
```

### Filter Events

Extract specific events:

```bash
# Filter by agent
sage trace filter trace.ndjson --agent Worker

# Filter by event kind
sage trace filter trace.ndjson --kind infer.complete

# Filter by time range
sage trace filter trace.ndjson --after 0.5 --before 1.0
```

Filtered output is valid NDJSON, so you can pipe it to other commands.

### LLM Call Analysis

View all LLM calls:

```bash
sage trace infer trace.ndjson
```

Output:
```
LLM Calls
─────────────────────────────────────────────────────────
Agent       Prompt                           Duration  Status
─────────────────────────────────────────────────────────
Worker      "Summarise: quantum computing"   826ms     OK
Worker      "Summarise: gene editing"        885ms     OK
─────────────────────────────────────────────────────────
Total: 2 calls, 1711ms, 100% success
```

### Cost Estimation

Estimate API costs (requires token counts in trace):

```bash
sage trace cost trace.ndjson
```

Output:
```
Estimated Cost
──────────────────────────────
Model: gpt-4o-mini
Input tokens:   1,234
Output tokens:  2,456
──────────────────────────────
Estimated: $0.0037
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `SAGE_TRACE` | Enable tracing (`1` or `true`) | Off |
| `SAGE_TRACE_FILE` | Write traces to this file | None |

## Best Practices

### 1. Use trace() for debugging

Add trace events at key points in your agent logic:

```sage
trace("Starting phase 1: data collection");
let data = try collect_data();
trace("Phase 1 complete, got {len(data)} items");
```

### 2. Analyse LLM performance

Use `sage trace infer` to identify slow or failing LLM calls:

```bash
sage trace infer trace.ndjson | sort -t$'\t' -k3 -rn
```

### 3. Compare runs

Save traces from different runs and compare:

```bash
sage run program.sg --trace-file before.ndjson
# ... make changes ...
sage run program.sg --trace-file after.ndjson

sage trace summary before.ndjson
sage trace summary after.ndjson
```

### 4. Filter for debugging

When debugging a specific agent:

```bash
sage trace filter trace.ndjson --agent MyAgent | sage trace pretty /dev/stdin
```

### 5. Monitor in production

Write traces to a file and use standard log aggregation tools:

```bash
sage run program.sg --trace-file /var/log/sage/trace.ndjson
```

The NDJSON format is compatible with tools like `jq`, Elasticsearch, and Datadog.
