# Error Handling

Sage has a robust error handling system designed for the realities of AI-native applications, where LLM calls can fail, agents can crash, and network operations are inherently unreliable.

## The Error Model

In Sage, errors are values. Operations that can fail are marked with `fails` and must be explicitly handled. This prevents silent failures and makes error paths visible in your code.

**Fallible operations in Sage:**
- `divine` — LLM calls
- `await` — waiting for agents
- `send` — sending messages to agents
- Functions marked with `fails`
- Tool calls (e.g., `Http.get`)

## Handling Errors with `try`

The `try` keyword propagates errors to the enclosing `on error` handler:

```sage
agent Researcher {
    topic: String

    on start {
        let summary = try divine("Summarise: {self.topic}");
        yield(summary);
    }

    on error(e) {
        print("Research failed: " ++ e.message);
        yield("Unable to research topic");
    }
}

run Researcher { topic: "quantum computing" };
```

When the `divine` call fails, execution jumps to `on error`. The error `e` contains:
- `message` — human-readable description
- `kind` — error category (see Error Kinds below)

## Inline Recovery with `catch`

For fine-grained control, use `catch` to handle errors inline:

```sage
agent Main {
    on start {
        let result = catch divine("What is 2+2?") {
            "I don't know"
        };
        print(result);
        yield(0);
    }
}

run Main;
```

If `divine` fails, the catch block runs and its value becomes the result. This is useful when you want to provide a fallback without involving the agent's error handler.

### Catch with Error Binding

You can bind the error to inspect it:

```sage
let result = catch divine("prompt") as err {
    print("Failed: " ++ err.message);
    "fallback value"
};
```

## Explicit Failure with `fail`

Use `fail` to raise errors explicitly:

```sage
fn validate_age(age: Int) -> Int fails {
    if age < 0 {
        fail "Age cannot be negative";
    }
    if age > 150 {
        fail "Age seems unrealistic";
    }
    return age;
}
```

The `fail` expression:
- Immediately returns an error from the current function
- The function must be marked with `fails`
- Takes a string message

## Retrying Operations

For transient failures, use `retry`:

```sage
agent Fetcher {
    url: String

    on start {
        // Retry up to 3 times
        let response = retry(3) {
            try Http.get(self.url)
        };
        yield(response.body);
    }

    on error(e) {
        yield("Failed after retries");
    }
}
```

### Retry with Delay

Add a delay between attempts:

```sage
let result = retry(3, delay: 1000) {
    try divine("Generate a haiku")
};
```

This waits 1000ms between each retry attempt.

### Retry with Error Filtering

Only retry on specific error kinds:

```sage
let result = retry(3, on: [ErrorKind.Network, ErrorKind.Timeout]) {
    try Http.get(url)
};
```

Other errors (like `ErrorKind.User`) will fail immediately without retrying.

## Error Kinds

Sage categorises errors into kinds for programmatic handling:

| Kind | Description | Examples |
|------|-------------|----------|
| `Llm` | LLM-related failures | API errors, parse failures, empty responses |
| `Agent` | Agent lifecycle errors | Spawn failures, await timeouts |
| `Runtime` | Internal runtime errors | Type mismatches |
| `Tool` | Tool call failures | HTTP errors, file I/O errors |
| `User` | User-raised errors | From `fail` expressions |

### Matching on Error Kind

```sage
on error(e) {
    match e.kind {
        ErrorKind.Llm => {
            print("LLM failed, using fallback");
            yield(fallback_response());
        }
        ErrorKind.Network => {
            print("Network issue, please retry");
            yield(1);
        }
        _ => {
            print("Unexpected error: " ++ e.message);
            yield(1);
        }
    }
}
```

## Fallible Functions

Mark functions that can fail with `fails`:

```sage
fn fetch_user(id: Int) -> User fails {
    let response = try Http.get("/users/" ++ str(id));
    if response.status != 200 {
        fail "User not found";
    }
    return parse_user(response.body);
}
```

Callers must handle the error:

```sage
// With try
let user = try fetch_user(42);

// With catch
let user = catch fetch_user(42) {
    User { name: "Unknown", id: 0 }
};
```

## Best Practices

### 1. Handle errors at the right level

Use `try` for errors that should bubble up to the agent's error handler. Use `catch` for errors you want to handle locally with a fallback.

### 2. Provide meaningful fallbacks

```sage
// Good: meaningful fallback
let summary = catch divine("Summarise: {topic}") {
    "Summary unavailable for " ++ topic
};

// Avoid: silent failures
let summary = catch divine("Summarise: {topic}") {
    ""
};
```

### 3. Use retry for transient failures

LLM calls and network requests often fail transiently. Use `retry` with appropriate delays:

```sage
let result = retry(3, delay: 500) {
    try divine("Generate response")
};
```

### 4. Log errors in on error

```sage
on error(e) {
    print("Error [" ++ str(e.kind) ++ "]: " ++ e.message);
    yield(error_response);
}
```

### 5. Fail fast on unrecoverable errors

```sage
fn validate_config(config: Config) -> Config fails {
    if is_empty(config.api_key) {
        fail "API key is required";
    }
    return config;
}
```

## Summary

| Construct | Purpose |
|-----------|---------|
| `try expr` | Propagate error to `on error` handler |
| `catch expr { fallback }` | Handle error inline with fallback |
| `fail "message"` | Raise an explicit error |
| `retry(n) { expr }` | Retry operation up to n times |
| `on error(e) { ... }` | Agent-level error handler |
| `fails` | Mark function as fallible |
