# Built-in Tools

Sage provides built-in tools that agents can use to interact with external services. Tools are declared with `use` and their methods are called with the `Tool.method()` syntax.

## Declaring Tools

To use a tool in an agent, declare it with `use` at the top of the agent body:

```sage
agent Fetcher {
    use Http

    on start {
        let response = try Http.get("https://httpbin.org/get");
        yield(response.status);
    }

    on error(e) {
        yield(-1);
    }
}

run Fetcher;
```

## Available Tools

| Tool | Description |
|------|-------------|
| [Http](./http.md) | HTTP client for web requests |
| [Database](./database.md) | SQL database client (SQLite, PostgreSQL, MySQL) |
| [Fs](./filesystem.md) | Filesystem operations (read, write, list, delete) |
| [Shell](./shell.md) | Execute shell commands |

## Error Handling

Tool calls are fallible operations. You must handle potential errors using `try` or `catch`:

```sage
// Propagate errors to the agent's on error handler
let response = try Http.get(url);

// Handle errors inline with a fallback
let response = catch Http.get(url) {
    HttpResponse { status: 0, body: "", headers: {} }
};
```

## Environment Configuration

Tools can be configured via environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `SAGE_HTTP_TIMEOUT` | HTTP request timeout in seconds | `30` |
| `SAGE_DATABASE_URL` | Database connection URL | (required for Database tool) |
| `SAGE_FS_ROOT` | Root directory for filesystem operations | `.` |
