# Database Client

The `Database` tool provides SQL query capabilities for agents. It supports SQLite, PostgreSQL, and MySQL via connection URLs.

## Usage

Declare the tool with `use Database` in your agent:

```sage
agent DataAgent {
    use Database

    on start {
        let rows = try Database.query("SELECT id, name FROM users");
        for row in rows {
            print(row.columns);  // ["id", "name"]
            print(row.values);   // ["1", "Alice"]
        }
        yield(0);
    }

    on error(e) {
        print("Database error");
        yield(-1);
    }
}

run DataAgent;
```

## Methods

### `Database.query(sql: String) -> List<DbRow>`

Executes a SELECT query and returns the results as a list of rows.

```sage
let rows = try Database.query("SELECT * FROM users WHERE active = true");
```

### `Database.execute(sql: String) -> Int`

Executes an INSERT, UPDATE, or DELETE statement and returns the number of affected rows.

```sage
let affected = try Database.execute(
    "INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com')"
);
print("Inserted: " ++ int_to_str(affected) ++ " rows");
```

## DbRow

Query results are returned as `DbRow` records:

| Field | Type | Description |
|-------|------|-------------|
| `columns` | `List<String>` | Column names from the query |
| `values` | `List<String>` | Values as strings |

## Configuration

| Variable | Description | Required |
|----------|-------------|----------|
| `SAGE_DATABASE_URL` | Database connection URL | Yes |

### Connection URL Formats

**SQLite:**
```bash
SAGE_DATABASE_URL="sqlite:./data.db"
SAGE_DATABASE_URL="sqlite::memory:"   # In-memory database
```

**PostgreSQL:**
```bash
SAGE_DATABASE_URL="postgres://user:password@localhost/dbname"
```

**MySQL:**
```bash
SAGE_DATABASE_URL="mysql://user:password@localhost/dbname"
```

## Examples

### CRUD Operations

```sage
agent UserManager {
    use Database

    on start {
        // Create table
        try Database.execute("
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY,
                name TEXT NOT NULL,
                email TEXT
            )
        ");

        // Insert
        let inserted = try Database.execute(
            "INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com')"
        );
        print("Inserted " ++ int_to_str(inserted) ++ " user");

        // Select
        let users = try Database.query("SELECT id, name, email FROM users");
        for user in users {
            print("User: " ++ user.values.1 ++ " <" ++ user.values.2 ++ ">");
        }

        // Update
        let updated = try Database.execute(
            "UPDATE users SET email = 'alice@newdomain.com' WHERE name = 'Alice'"
        );
        print("Updated " ++ int_to_str(updated) ++ " user");

        // Delete
        let deleted = try Database.execute("DELETE FROM users WHERE id = 1");
        print("Deleted " ++ int_to_str(deleted) ++ " user");

        yield(0);
    }

    on error(e) {
        yield(-1);
    }
}

run UserManager;
```

### Querying with Aggregates

```sage
agent StatsAgent {
    use Database

    on start {
        let stats = try Database.query("
            SELECT
                COUNT(*) as total,
                AVG(age) as avg_age
            FROM users
        ");

        if len(stats) > 0 {
            print("Total users: " ++ stats.0.values.0);
            print("Average age: " ++ stats.0.values.1);
        }

        yield(0);
    }

    on error(e) {
        yield(-1);
    }
}

run StatsAgent;
```

## Notes

- SQL queries are executed directly; be careful with user input to prevent SQL injection
- Values are returned as strings; use `parse_int()` or similar to convert numeric values
- The `database` feature must be enabled at compile time (it is by default)
