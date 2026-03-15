# Built-in Tools

Sage provides built-in tools that agents can use to interact with external services. Tools are declared with `use` and their methods are called with the `Tool.method()` syntax.

## Declaring Tools

To use a tool in an agent, declare it with `use` at the top of the agent body:

```sage
agent Fetcher {
    use Http

    on start {
        let response = try Http.get("https://httpbin.org/get");
        emit(response.status);
    }

    on error(e) {
        emit(-1);
    }
}

run Fetcher;
```

## Available Tools

| Tool | Description |
|------|-------------|
| [Http](./http.md) | HTTP client for web requests |

More tools are planned for future releases (Fs, Kv, Browser, etc.).

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
