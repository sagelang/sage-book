# MCP Integration

> **New in v2.2.0** — RFC-0023

Sage supports the [Model Context Protocol](https://modelcontextprotocol.io/) (MCP) for connecting agents to external tool servers. There are two complementary modes:

1. **Typed MCP Tools** — compile-time checked tool interfaces backed by MCP servers
2. **Dynamic MCP** — runtime tool discovery and invocation for orchestration scenarios

## Typed MCP Tools

Declare MCP tools using the `tool` keyword, exactly like built-in tools:

```sage
tool Github {
    fn search_repositories(query: String) -> String
    fn list_issues(owner: String, repo: String) -> String
    fn get_issue(owner: String, repo: String, issue_number: Int) -> String
    fn create_issue(owner: String, repo: String, title: String, body: String) -> String
}
```

Tool functions are implicitly fallible — you must use `try` or `catch` when calling them. Parameter and return types map directly to JSON Schema for MCP serialization.

### Using MCP Tools in Agents

Agents declare MCP tool usage with `use` statements, identical to built-in tools:

```sage
agent IssueScanner {
    use Github

    owner: String
    repo: String

    on start {
        let raw = try Github.list_issues(self.owner, self.repo);

        let summary = try divine(
            "Summarise these issues: {raw}"
        );

        yield(summary);
    }

    on error(e) {
        yield("Unavailable");
    }
}
```

### Tool Name Mapping

If the MCP server uses different naming conventions (e.g. kebab-case), use the `#[mcp_name]` attribute:

```sage
tool Github {
    #[mcp_name = "create-issue"]
    fn create_issue(repo: String, title: String, body: String) -> String

    #[mcp_name = "list-issues"]
    fn list_issues(repo: String, state: String) -> String
}
```

### Type Mapping

Arguments are serialized to JSON objects. Return values are deserialized from tool results.

| Sage Type | JSON Schema | Example |
|-----------|------------|---------|
| `Int` | `integer` | `42` |
| `Float` | `number` | `3.14` |
| `Bool` | `boolean` | `true` |
| `String` | `string` | `"hello"` |
| `List<T>` | `array` | `[1, 2, 3]` |
| `Map<String, V>` | `object` | `{"a": 1}` |
| `Option<T>` | nullable T | `42` or `null` |
| `record Foo { x: Int }` | `object` | `{"x": 42}` |
| `enum Status { Active }` | `string` | `"Active"` |

Result deserialization:
1. If the MCP response has `structuredContent` matching the return type schema, it is deserialized directly
2. If the response has a single text content item, JSON deserialization is attempted
3. If the return type is `String`, the text value is used directly

## Configuration

MCP servers are configured in `grove.toml` using `[tools.X]` sections.

### Stdio Transport

Launch the server as a subprocess:

```toml
[tools.Github]
transport = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
timeout_ms = 30000
connect_timeout_ms = 10000

[tools.Github.env]
GITHUB_PERSONAL_ACCESS_TOKEN = "$GITHUB_TOKEN"
```

| Field | Default | Description |
|-------|---------|-------------|
| `transport` | — | `"stdio"` for subprocess servers |
| `command` | — | Executable to launch |
| `args` | `[]` | Command arguments |
| `timeout_ms` | `30000` | Per-call timeout in milliseconds |
| `connect_timeout_ms` | `10000` | Connection timeout in milliseconds |

Environment variables in the `[tools.X.env]` section starting with `$` are resolved from the host environment.

### HTTP Transport

Connect to a remote MCP server:

```toml
[tools.Slack]
transport = "http"
url = "https://mcp.slack.example.com/mcp"
timeout_ms = 30000
```

#### Bearer Token Auth

```toml
[tools.API]
transport = "http"
url = "https://api.example.com/mcp"
auth = "bearer"
token_env = "API_TOKEN"
```

#### OAuth 2.1 + PKCE

```toml
[tools.CloudAPI]
transport = "http"
url = "https://cloud.example.com/mcp"
auth = "oauth"
client_id_env = "CLOUD_CLIENT_ID"
authorization_url = "https://auth.cloud.example.com/authorize"
token_url = "https://auth.cloud.example.com/token"
scopes = ["tools:read", "tools:write"]
```

## Dynamic MCP

For scenarios where tools aren't known at compile time, use the dynamic MCP functions:

```sage
agent DynamicExplorer {
    config_json: String

    on start {
        let handle = try mcp_connect(self.config_json);

        let tools = try mcp_list_tools(handle);

        let args = '{"repo": "sagelang/sage", "state": "open"}';
        let result = try mcp_call(handle, "list-issues", args);

        try mcp_disconnect(handle);

        yield(result);
    }

    on error(e) {
        yield("failed");
    }
}
```

### Dynamic MCP Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `mcp_connect` | `(String) -> McpConnection fails` | Connect using a JSON config string |
| `mcp_list_tools` | `(McpConnection) -> List<McpTool> fails` | List available tools |
| `mcp_call` | `(McpConnection, String, String) -> String fails` | Call a tool with JSON args |
| `mcp_call_json` | `(McpConnection, String, Map<String, String>) -> String fails` | Call a tool with a Map |
| `mcp_disconnect` | `(McpConnection) -> Unit fails` | Disconnect from the server |
| `mcp_server_info` | `(McpConnection) -> McpServerInfo fails` | Get server metadata |

### Dynamic MCP Types

```sage
record McpConnection { id: Int }
record McpTool { name: String, description: String, input_schema: String }
record McpServerInfo { name: String, version: String }
```

## Testing MCP Tools

MCP tools integrate with the existing mock system:

```sage
test "issue filing works" {
    mock tool Github.create_issue -> '{"number": 42, "url": "..."}';

    let agent = summon IssueFiler { title: "Test", body: "Body" };
    let result = try await agent;
    assert_eq(result, 42);
}

test "handles server failure" {
    mock tool Github.create_issue -> fail("Server unavailable");

    let agent = summon IssueFiler { title: "Test", body: "Body" };
    let result = try await agent;
    assert_eq(result, -1);
}
```

Mocks intercept before the MCP transport layer. Dynamic MCP calls can also be mocked with `mock tool mcp.call -> "json"`.

## CLI Commands

```bash
# List configured MCP tools
sage tools list

# Inspect a server's tool manifest
sage tools inspect --stdio "npx -y @modelcontextprotocol/server-github"
sage tools inspect --http "https://mcp.example.com/mcp"

# Generate Sage tool declarations from a server
sage tools generate --stdio "npx -y @modelcontextprotocol/server-github" -o src/tools/github.sg

# Verify declared signatures match the server
sage check --verify-tools
```

## Error Codes

| Code | Condition |
|------|-----------|
| E080 | Agent uses a `tool` with no `[tools.X]` in `grove.toml` and it's not built-in |
| E081 | `[tools.X]` section missing required fields |
| E082 | (with `--verify-tools`) Declared signature doesn't match server manifest |
| E083 | `#[mcp_name]` attribute value isn't a string literal |

## Complete Example

**grove.toml:**

```toml
[project]
name = "mcp-devops"
entry = "src/main.sg"

[tools.Github]
transport = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
timeout_ms = 30000

[tools.Github.env]
GITHUB_PERSONAL_ACCESS_TOKEN = "$GITHUB_TOKEN"

[tools.Filesystem]
transport = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/tmp/sage-devops"]

[persistence]
backend = "sqlite"
path = ".sage/devops_state.db"
```

**src/main.sg:**

```sage
tool Github {
    fn list_issues(owner: String, repo: String) -> String
    fn create_issue(owner: String, repo: String, title: String, body: String) -> String
}

tool Filesystem {
    fn write_file(path: String, content: String) -> String
    fn read_file(path: String) -> String
}

agent IssueScanner {
    use Github

    owner: String
    repo: String

    on start {
        let raw = try Github.list_issues(self.owner, self.repo);
        let summary = try divine("Summarise these issues: {raw}");
        yield(summary);
    }

    on error(e) {
        yield("Unavailable");
    }
}

agent ReportWriter {
    use Filesystem

    issues: String

    on start {
        let report = try divine(
            "Write a markdown report from these issues:\n{self.issues}"
        );
        try Filesystem.write_file("/tmp/sage-devops/report.md", report);
        yield(report);
    }

    on error(e) {
        yield("Report generation failed");
    }
}

agent Coordinator {
    on start {
        let scanner = summon IssueScanner {
            owner: "sagelang",
            repo: "sage"
        };
        let issues = try await scanner;

        let writer = summon ReportWriter { issues: issues };
        let report = try await writer;

        print(report);
        yield(0);
    }

    on error(e) {
        print("Pipeline failed: " ++ str(e));
        yield(1);
    }
}

run Coordinator;
```
