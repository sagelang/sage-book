# Testing Overview

Sage has a built-in testing framework that makes it easy to test your agents and functions. Tests are first-class citizens in the language, not bolted-on annotations.

## Why Built-In Testing?

Agent-based systems are notoriously hard to test:
- LLM calls are non-deterministic
- Agent lifecycles involve async operations
- Message passing creates complex interaction patterns

Sage's testing framework solves these problems with:
- **First-class LLM mocking** — deterministic tests without network calls
- **Async-aware test bodies** — `spawn` and `await` work naturally in tests
- **Concurrent execution** — tests run in parallel by default for speed

## Quick Start

Create a test file ending in `_test.sg`:

**src/math_test.sg:**
```sage
test "addition works" {
    assert_eq(1 + 1, 2);
}

test "multiplication works" {
    let result = 6 * 7;
    assert_eq(result, 42);
}
```

Run your tests:
```bash
sage test .
```

Output:
```
🦉 Ward Running 2 tests from 1 file

  PASS math_test.sg::addition works
  PASS math_test.sg::multiplication works

🦉 Ward test result: ok. 2 passed, 0 failed, 0 skipped [0.82s]
```

## Test File Convention

Test files must end in `_test.sg`. The test runner automatically discovers all test files in your project:

```
my_project/
├── grove.toml
└── src/
    ├── main.sg
    ├── utils.sg
    ├── utils_test.sg    # Tests for utils.sg
    └── agents_test.sg   # Tests for agents
```

## Next Steps

- [Writing Tests](./writing-tests.md) — test syntax and best practices
- [Assertions](./assertions.md) — available assertion functions
- [Mocking LLMs](./mocking.md) — how to mock `infer` calls
