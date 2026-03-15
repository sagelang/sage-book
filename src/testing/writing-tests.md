# Writing Tests

## Test Syntax

Tests are declared with the `test` keyword followed by a description string and a block:

```sage
test "descriptive name for the test" {
    // test body
}
```

The description appears in test output, so make it meaningful:
- ✓ `"user can log in with valid credentials"`
- ✓ `"empty list returns None for find"`
- ✗ `"test1"` (not descriptive)

## Serial Tests

By default, tests run concurrently for speed. Use `@serial` when a test needs isolation:

```sage
@serial test "modifies global state" {
    // This test runs alone, not concurrently with others
}
```

Use `@serial` when:
- Tests modify shared state
- Tests depend on specific timing
- Tests use resources that can't be shared

## Testing Functions

Test regular functions by calling them and asserting on results:

```sage
fn factorial(n: Int) -> Int {
    if n <= 1 {
        return 1;
    }
    return n * factorial(n - 1);
}

test "factorial of 5 is 120" {
    assert_eq(factorial(5), 120);
}

test "factorial of 0 is 1" {
    assert_eq(factorial(0), 1);
}

test "factorial of 1 is 1" {
    assert_eq(factorial(1), 1);
}
```

## Testing Agents

Test agents by spawning them with mocked LLM responses:

```sage
agent Summariser {
    topic: String

    on start {
        let summary = try infer("Summarise: {self.topic}");
        emit(summary);
    }

    on error(e) {
        emit("Error occurred");
    }
}

test "summariser returns LLM response" {
    mock infer -> "This is a summary of quantum physics.";

    let result = await spawn Summariser { topic: "quantum physics" };
    assert_eq(result, "This is a summary of quantum physics.");
}
```

## Test Body Semantics

Test bodies are async by default — you can use `await` and `spawn` without special syntax:

```sage
test "two agents can run concurrently" {
    mock infer -> "Result A";
    mock infer -> "Result B";

    let a = spawn Researcher { topic: "A" };
    let b = spawn Researcher { topic: "B" };

    let result_a = await a;
    let result_b = await b;

    assert_eq(result_a, "Result A");
    assert_eq(result_b, "Result B");
}
```

## Organising Tests

Keep tests close to the code they test:

```
src/
├── auth.sg
├── auth_test.sg      # Tests for auth.sg
├── payments.sg
└── payments_test.sg  # Tests for payments.sg
```

Or use a dedicated test directory:

```
src/
├── main.sg
└── lib/
    ├── utils.sg
    └── utils_test.sg
```
