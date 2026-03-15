# Mocking LLM Calls

The most powerful feature of Sage's testing framework is first-class LLM mocking. In test files, you can specify exactly what `infer` calls should return, making your tests deterministic and fast.

## Basic Mocking

Use `mock infer -> value;` to specify what the next `infer` call should return:

```sage
test "infer returns mocked value" {
    mock infer -> "This is a mocked response";

    let result: String = try infer("Summarise something");
    assert_eq(result, "This is a mocked response");
}
```

The mock is consumed by the `infer` call — each mock is used exactly once.

## Multiple Mocks

When your test makes multiple `infer` calls, queue up multiple mocks in order:

```sage
test "multiple infer calls" {
    mock infer -> "First response";
    mock infer -> "Second response";
    mock infer -> "Third response";

    let r1 = try infer("Query 1");
    let r2 = try infer("Query 2");
    let r3 = try infer("Query 3");

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
    mock infer -> Summary {
        text: "Quantum computing is fast.",
        confidence: 0.88
    };

    let summary: Summary = try infer("Summarise quantum computing");
    assert_eq(summary.text, "Quantum computing is fast.");
    assert_gt(summary.confidence, 0.8);
}
```

## Mocking Failures

Use `fail("message")` to mock an `infer` failure:

```sage
test "agent handles infer failure" {
    mock infer -> fail("rate limit exceeded");

    let handle = spawn ResilientResearcher { topic: "test" };
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
        let summary = try infer("Research: {self.topic}");
        emit(summary);
    }

    on error(e) {
        emit("Research failed");
    }
}

test "researcher emits summary" {
    mock infer -> "Quantum computing uses qubits.";

    let result = await spawn Researcher { topic: "quantum" };
    assert_eq(result, "Quantum computing uses qubits.");
}
```

## Testing Multi-Agent Systems

For agents that spawn other agents, each agent's `infer` calls consume mocks in execution order:

```sage
test "coordinator gets results from two researchers" {
    mock infer -> "Summary about AI";
    mock infer -> "Summary about robots";

    let c = spawn Coordinator {
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

## Best Practices

1. **One assertion per test** — easier to identify failures
2. **Descriptive mock values** — make it clear what's being tested
3. **Test error paths** — use `fail()` to test error handling
4. **Keep mocks simple** — avoid complex JSON in mocks when possible
