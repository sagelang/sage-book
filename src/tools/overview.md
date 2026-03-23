# Built-in Tools

Sage provides built-in tools that agents can use to interact with external services: databases, HTTP APIs, filesystems, and shell commands. Tools are **capability declarations** — an agent must explicitly declare which tools it uses, making its external interactions visible in its signature.

## Declaring Tool Usage

Use the `use` keyword inside an agent to declare which tools it needs:

```sage
agent DataFetcher {
    use Http
    use Database

    on start {
        // Both Http and Database methods are now available
        let response = try Http.get("https://api.example.com/status");
        let rows = try Database.query("SELECT * FROM cache");
        yield(0);
    }

    on error(e) {
        yield(-1);
    }
}

run DataFetcher;
```

Attempting to use a tool method without declaring it is a compile error:

```sage
agent Broken {
    // No `use Http` declaration

    on start {
        let r = try Http.get("...");  // Error E038: undeclared tool use
        yield(0);
    }
}
```

This is intentional. The `use` clause is a **capability declaration** — it makes an agent's external interactions explicit and auditable.

## Available Tools

| Tool | Description | Methods |
|------|-------------|---------|
| [Http](./http.md) | HTTP client for web requests | `get`, `post`, `put`, `delete` |
| [Database](./database.md) | SQL database client | `query`, `execute` |
| [Fs](./filesystem.md) | Filesystem operations | `read`, `write`, `exists`, `list`, `delete` |
| [Shell](./shell.md) | Execute shell commands | `run` |

## Tool Calls Are Fallible

Every tool call can fail — network timeouts, database connection errors, file not found, command failures. Tool methods return `Result<T, ToolError>` implicitly, so you must handle errors.

### Using `try`

The `try` keyword unwraps the result and propagates errors to the agent's `on error` handler:

```sage
agent Fetcher {
    use Http

    on start {
        // If this fails, control jumps to on error
        let response = try Http.get("https://api.example.com/data");
        print("Got: " ++ response.body);
        yield(response.status);
    }

    on error(e) {
        print("Request failed: " ++ e.message);
        yield(-1);
    }
}
```

### Using `catch`

The `catch` expression provides a fallback value when the call fails:

```sage
let response = catch Http.get(url) {
    HttpResponse { status: 0, body: "", headers: {} }
};

if response.status == 0 {
    print("Request failed, using fallback");
}
```

### Using `match`

For fine-grained control, call without `try` and match on the result:

```sage
let result = Http.get(url);
match result {
    Ok(response) => {
        print("Success: " ++ response.body);
    }
    Err(e) => {
        print("Failed: " ++ e.message);
        // Retry logic, logging, etc.
    }
}
```

## Configuration

Tools can be configured in two ways: environment variables (simple) or `grove.toml` (recommended for projects).

### Environment Variables

Quick configuration for development:

```bash
# HTTP
export SAGE_HTTP_TIMEOUT=60

# Database
export SAGE_DATABASE_URL="postgres://localhost/myapp"

# Filesystem
export SAGE_FS_ROOT="/var/data"
```

### grove.toml Configuration

For projects, configure tools in your `grove.toml`:

```toml
[project]
name = "my-steward"

[tools.database]
driver = "postgres"
url = "postgresql://user:pass@localhost/myapp"
pool_size = 10

[tools.http]
timeout_ms = 30000

[tools.filesystem]
root = "./data"
```

#### Database Configuration

```toml
[tools.database]
driver = "postgres"       # postgres | sqlite | mysql
url = "postgresql://..."  # Connection URL
pool_size = 5             # Connection pool size (default: 5)
```

#### HTTP Configuration

```toml
[tools.http]
timeout_ms = 30000        # Request timeout (default: 30000)
```

#### Filesystem Configuration

```toml
[tools.filesystem]
root = "./data"           # All paths relative to this root
```

## Multiple Tools

Declare multiple tools by listing them separately:

```sage
agent FullStack {
    use Http
    use Database
    use Fs
    use Shell

    on start {
        // Fetch data from API
        let api_data = try Http.get("https://api.example.com/data");

        // Store in database
        try Database.execute("INSERT INTO cache (data) VALUES ('{api_data.body}')");

        // Write to file
        try Fs.write("cache/latest.json", api_data.body);

        // Run a post-processing script
        let result = try Shell.run("./scripts/process.sh");

        yield(result.exit_code);
    }

    on error(e) {
        yield(-1);
    }
}
```

## Tool Result Types

Each tool has specific return types for its methods:

### HttpResponse

```sage
record HttpResponse {
    status: Int,
    body: String,
    headers: Map<String, String>,
}
```

### DbRow

```sage
record DbRow {
    columns: List<String>,
    values: List<String>,
}
```

### ShellResult

```sage
record ShellResult {
    exit_code: Int,
    stdout: String,
    stderr: String,
}
```

## Testing with Mock Tools

In test files (`*_test.sg`), you can mock tool responses:

```sage
test "handles API response" {
    mock tool Http.get -> HttpResponse {
        status: 200,
        body: "{\"user\": \"alice\"}",
        headers: {}
    };

    // Agent under test will receive the mocked response
    let agent = summon DataFetcher {};
    let result = await(agent);
    assert_eq(result, 200);
}

test "handles API failure" {
    mock tool Http.get -> fail("connection refused");

    let agent = summon DataFetcher {};
    let result = await(agent);
    assert_eq(result, -1);  // Error handler returns -1
}
```

See [Testing > Mocking](../testing/mocking.md) for details.

## Best Practices

1. **Declare only what you need.** Don't add `use Shell` unless you actually run commands. The capability list should be minimal.

2. **Always handle errors.** Tool calls fail in production. Use `try` with a robust `on error` handler, or `catch` with sensible defaults.

3. **Configure via grove.toml.** Environment variables work but grove.toml is versioned and explicit.

4. **Be careful with Shell.** Arbitrary command execution is powerful but dangerous. Validate inputs, avoid string interpolation with untrusted data.

5. **Test with mocks.** Don't hit real databases or APIs in tests. Mock tool responses for reliable, fast tests.
