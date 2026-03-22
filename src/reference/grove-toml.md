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

[extern]
modules = ["src/sage_extern.rs"]

[extern.dependencies]
chrono = "0.4"
```
