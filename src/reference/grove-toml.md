# grove.toml Reference

The `grove.toml` file is the project manifest for Sage projects. It configures the project name, entry point, dependencies, persistence, supervision, and extern functions.

## [project]

Basic project metadata.

```toml
[project]
name = "my_project"
entry = "src/main.sg"
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Project name (used for the generated binary) |
| `entry` | Yes | Path to the entry point `.sg` file |

## [dependencies]

Git-based or local path dependencies for multi-package projects.

```toml
[dependencies]
mylib = { git = "https://github.com/user/mylib" }
utils = { git = "https://github.com/user/utils", tag = "v1.0.0" }
local-lib = { path = "../shared-lib" }
```

| Field | Description |
|-------|-------------|
| `git` | Git repository URL |
| `path` | Local path (relative to project root) |
| `tag` | Git tag |
| `branch` | Git branch |
| `rev` | Git commit SHA |

Manage dependencies with `sage add` and `sage update`.

## [persistence]

Configure automatic checkpointing for `@persistent` agent fields.

```toml
[persistence]
backend = "sqlite"
path = ".sage/checkpoints.db"
```

| Field | Default | Description |
|-------|---------|-------------|
| `backend` | `"sqlite"` | Storage backend: `"sqlite"`, `"postgres"`, `"file"` |
| `path` | `".sage/checkpoints.db"` | Path for SQLite/file backends |
| `url` | — | Connection URL for PostgreSQL backend |

### Backend examples

```toml
# SQLite (default)
[persistence]
backend = "sqlite"
path = ".sage/checkpoints.db"

# PostgreSQL
[persistence]
backend = "postgres"
url = "postgres://user:password@localhost/mydb"

# File-based (JSON files)
[persistence]
backend = "file"
path = ".sage/state"
```

## [supervision]

Configure supervision tree parameters.

```toml
[supervision]
max_restarts = 5
restart_window_s = 60
```

| Field | Default | Description |
|-------|---------|-------------|
| `max_restarts` | `3` | Maximum restarts before circuit breaker trips |
| `restart_window_s` | `5` | Time window (seconds) for counting restarts |

When `max_restarts` is exceeded within `restart_window_s`, the supervisor stops all children and shuts down.

## [extern]

Configure Rust FFI for extern function declarations.

```toml
[extern]
modules = ["src/sage_extern.rs"]

[extern.dependencies]
chrono = "0.4"
reqwest = { version = "0.12", features = ["blocking"] }
```

| Field | Description |
|-------|-------------|
| `modules` | List of Rust source files to compile and link |

### [extern.dependencies]

Additional Cargo dependencies needed by your extern Rust code. Uses standard Cargo dependency syntax:

```toml
[extern.dependencies]
# Simple version
serde = "1.0"

# With features
tokio = { version = "1", features = ["full"] }

# Git dependency
my-crate = { git = "https://github.com/user/crate" }
```

These are added to the generated Cargo.toml alongside `sage-runtime`.

## [tools.X]

Configure MCP (Model Context Protocol) tool servers. Each tool gets its own `[tools.X]` section where `X` matches the `tool` declaration name in your Sage code.

### Stdio Transport

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

Environment variables in `[tools.X.env]` starting with `$` are resolved from the host environment.

### HTTP Transport

```toml
[tools.Slack]
transport = "http"
url = "https://mcp.slack.example.com/mcp"
timeout_ms = 30000
auth = "bearer"
token_env = "SLACK_MCP_TOKEN"
```

| Field | Default | Description |
|-------|---------|-------------|
| `transport` | — | `"http"` for remote servers |
| `url` | — | Server endpoint URL |
| `auth` | — | `"bearer"` or `"oauth"` |
| `token_env` | — | Environment variable name for bearer token |
| `client_id_env` | — | Environment variable for OAuth client ID |
| `authorization_url` | — | OAuth authorization endpoint |
| `token_url` | — | OAuth token endpoint |
| `scopes` | `[]` | OAuth scopes |

See [MCP Integration](../tools/mcp.md) for full documentation.

## Complete Example

```toml
[project]
name = "webapp_steward"
entry = "src/main.sg"

[dependencies]
shared = { path = "../shared-lib" }

[persistence]
backend = "sqlite"
path = ".sage/checkpoints.db"

[supervision]
max_restarts = 5
restart_window_s = 60

[tools.Github]
transport = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]

[tools.Github.env]
GITHUB_PERSONAL_ACCESS_TOKEN = "$GITHUB_TOKEN"

[extern]
modules = ["src/sage_extern.rs"]

[extern.dependencies]
chrono = "0.4"
```
