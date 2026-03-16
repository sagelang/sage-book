# Mocking

Sage's testing framework provides first-class mocking for both LLM calls and tool calls. This makes your tests deterministic, fast, and independent of external services.

## Mocking LLM Calls

You can specify exactly what `infer` calls should return using `mock divine`.

## Basic Mocking

Use `mock divine -> value;` to specify what the next `infer` call should return:

```sage
test "infer returns mocked value" {
    mock divine -> "This is a mocked response";

    let result: String = try divine("Summarise something");
    assert_eq(result, "This is a mocked response");
}
```

The mock is consumed by the `infer` call — each mock is used exactly once.

## Multiple Mocks

When your test makes multiple `infer` calls, queue up multiple mocks in order:

```sage
test "multiple infer calls" {
    mock divine -> "First response";
    mock divine -> "Second response";
    mock divine -> "Third response";

    let r1 = try divine("Query 1");
    let r2 = try divine("Query 2");
    let r3 = try divine("Query 3");

    assert_eq(r1, "First response");
    assert_eq(r2, "Second response");
    assert_eq(r3, "Third response");
}
```

Mocks are consumed in FIFO order (first in, first out).

## Mocking Structured Output

For typed `infer` calls, mock with the appropriate record structure:

```sage
record Summary {
    text: String,
    confidence: Float,
}

test "structured infer returns typed mock" {
    mock divine -> Summary {
        text: "Quantum computing is fast.",
        confidence: 0.88
    };

    let summary: Summary = try divine("Summarise quantum computing");
    assert_eq(summary.text, "Quantum computing is fast.");
    assert_gt(summary.confidence, 0.8);
}
```

## Mocking Failures

Use `fail("message")` to mock an `infer` failure:

```sage
test "agent handles infer failure" {
    mock divine -> fail("rate limit exceeded");

    let handle = summon ResilientResearcher { topic: "test" };
    let result = await handle;

    // Agent's fallback behaviour
    assert_eq(result, "unavailable");
}
```

This is essential for testing error handling paths.

## Testing Agents with Mocks

When testing agents that use `infer`, mocks are consumed by the agent's `infer` calls:

```sage
agent Researcher {
    topic: String

    on start {
        let summary = try divine("Research: {self.topic}");
        yield(summary);
    }

    on error(e) {
        yield("Research failed");
    }
}

test "researcher emits summary" {
    mock divine -> "Quantum computing uses qubits.";

    let result = await summon Researcher { topic: "quantum" };
    assert_eq(result, "Quantum computing uses qubits.");
}
```

## Testing Multi-Agent Systems

For agents that summon other agents, each agent's `infer` calls consume mocks in execution order:

```sage
test "coordinator gets results from two researchers" {
    mock divine -> "Summary about AI";
    mock divine -> "Summary about robots";

    let c = summon Coordinator {
        topics: ["AI", "robots"]
    };
    let results = await c;

    assert_contains(results, "AI");
    assert_contains(results, "robots");
}
```

## Mock Queue Exhaustion

If an `infer` call is made without an available mock, the test fails with error code E054:

```
Error: infer called with no mock available (E054)
```

Always provide enough mocks for all `infer` calls in your test.

## Mocking Tool Calls

Just like LLM calls, you can mock tool calls (Http, Fs, etc.) to avoid real network or filesystem operations in tests.

### Basic Tool Mocking

Use `mock tool ToolName.method -> value;` to specify what a tool call should return:

```sage
test "http get returns mocked response" {
    mock tool Http.get -> HttpResponse {
        status: 200,
        body: "Hello, World!",
        headers: {}
    };

    let response = try Http.get("https://example.com");
    assert_eq(response.status, 200);
    assert_eq(response.body, "Hello, World!");
}
```

### Mocking Tool Failures

Use `fail("message")` to mock a tool failure:

```sage
test "handles network error gracefully" {
    mock tool Http.get -> fail("connection timeout");

    let result = catch Http.get("https://example.com");
    assert_true(result.is_err());
}
```

### Multiple Tool Mocks

Like `mock divine`, tool mocks are consumed in FIFO order:

```sage
test "multiple http calls" {
    mock tool Http.get -> HttpResponse { status: 200, body: "first", headers: {} };
    mock tool Http.get -> HttpResponse { status: 200, body: "second", headers: {} };

    let r1 = try Http.get("https://api.example.com/1");
    let r2 = try Http.get("https://api.example.com/2");

    assert_eq(r1.body, "first");
    assert_eq(r2.body, "second");
}
```

### Mocking Different Tools

You can mock different tools in the same test:

```sage
test "agent uses multiple tools" {
    mock tool Http.get -> HttpResponse { status: 200, body: "data", headers: {} };
    mock tool Fs.read -> "config content";
    mock divine -> "processed result";

    let result = await summon DataProcessor {};
    assert_eq(result, "processed result");
}
```

### Testing Agents with Tool Mocks

When testing agents that use tools, mocks are consumed by the agent's tool calls:

```sage
agent Fetcher {
    url: String

    use Http

    on start {
        let response = try Http.get(self.url);
        yield(response.body);
    }
}

test "fetcher returns body" {
    mock tool Http.get -> HttpResponse {
        status: 200,
        body: "fetched content",
        headers: {}
    };

    let result = await summon Fetcher { url: "https://example.com" };
    assert_eq(result, "fetched content");
}
```

## Best Practices

1. **One assertion per test** — easier to identify failures
2. **Descriptive mock values** — make it clear what's being tested
3. **Test error paths** — use `fail()` to test error handling
4. **Keep mocks simple** — avoid complex JSON in mocks when possible
5. **Mock all external calls** — both `infer` and tool calls should be mocked for deterministic tests
