# Agent State

Agent fields are private state. They're initialized when the agent is spawned and can be accessed throughout the agent's lifetime.

## Declaring Fields

Agent state uses record-style field declarations:

```sage
agent Person {
    name: String
    age: Int
}
```

Fields must have explicit type annotations.

## Initializing Fields

When spawning an agent, provide values for all fields:

```sage
let p = spawn Person { name: "Alice", age: 30 };
```

Missing fields cause a compile error:

```sage
// Error: missing field `age` in spawn
let p = spawn Person { name: "Alice" };
```

## Accessing Fields

Use `self.fieldName` inside the agent:

```sage
agent Greeter {
    name: String

    on start {
        print("Hello, " ++ self.name ++ "!");
        emit(0);
    }
}
```

## Fields Are Immutable

Fields cannot be reassigned after initialization:

```sage
agent Counter {
    count: Int

    on start {
        // This won't work — fields are immutable
        // self.count = self.count + 1;

        // Use a local variable instead
        let count = self.count;
        count = count + 1;
        emit(count);
    }
}
```

## Entry Agent Fields

The entry agent (the one in `run`) cannot have required fields:

```sage
// Error: entry agent cannot have required fields
agent Main {
    config: String

    on start {
        emit(0);
    }
}

run Main;  // How would we provide `config`?
```

## Design Pattern: Configuration

Use fields to configure agent behavior:

```sage
agent Fetcher {
    url: String
    timeout: Int

    on start {
        // Use self.url and self.timeout
        emit("done");
    }
}

agent Main {
    on start {
        let f1 = spawn Fetcher {
            url: "https://api.example.com/a",
            timeout: 5000
        };
        let f2 = spawn Fetcher {
            url: "https://api.example.com/b",
            timeout: 3000
        };

        let r1 = try await f1;
        let r2 = try await f2;
        emit(0);
    }

    on error(e) {
        emit(1);
    }
}

run Main;
```
