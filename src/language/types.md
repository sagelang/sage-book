# Types

Sage has a simple but expressive type system.

## Primitive Types

| Type | Description | Example |
|------|-------------|---------|
| `Int` | 64-bit signed integer | `42`, `-17` |
| `Float` | 64-bit floating point | `3.14`, `-0.5` |
| `Bool` | Boolean | `true`, `false` |
| `String` | UTF-8 string | `"hello"` |
| `Unit` | No value (like Rust's `()`) | — |

## Compound Types

### List\<T\>

Ordered collection of elements:

```sage
let numbers: List<Int> = [1, 2, 3];
let names: List<String> = ["Alice", "Bob"];
let empty: List<Int> = [];
```

### Map\<K, V\>

Key-value collections:

```sage
let ages: Map<String, Int> = {"alice": 30, "bob": 25};
let alice_age = map_get(ages, "alice");  // Option<Int>

map_set(ages, "charlie", 35);
let has_bob = map_has(ages, "bob");      // true
let keys = map_keys(ages);               // List<String>
```

### Tuples

Fixed-size heterogeneous collections:

```sage
let pair: (Int, String) = (42, "hello");
let first = pair.0;   // 42
let second = pair.1;  // "hello"

// Tuple destructuring
let (x, y) = pair;

// Three-element tuple
let triple: (Int, String, Bool) = (1, "test", true);
```

### Option\<T\>

Optional values:

```sage
let some_value: Option<Int> = Some(42);
let no_value: Option<Int> = None;

// Pattern matching on Option
match some_value {
    Some(n) => print("Got: " ++ str(n)),
    None => print("Nothing"),
}
```

### Result\<T, E\>

Success or error values:

```sage
let success: Result<Int, String> = Ok(42);
let failure: Result<Int, String> = Err("not found");

match success {
    Ok(value) => print("Value: " ++ str(value)),
    Err(msg) => print("Error: " ++ msg),
}
```

### Fn(A, B) -> C

Function types for closures and higher-order functions:

```sage
let add: Fn(Int, Int) -> Int = |x: Int, y: Int| x + y;
let double: Fn(Int) -> Int = |x: Int| x * 2;

fn apply(f: Fn(Int) -> Int, x: Int) -> Int {
    return f(x);
}

let result = apply(double, 21);  // 42
```

## User-Defined Types

### Records

Define structured data with named fields:

```sage
record Point {
    x: Int,
    y: Int,
}

record Person {
    name: String,
    age: Int,
}
```

Construct records and access fields:

```sage
let p = Point { x: 10, y: 20 };
let sum = p.x + p.y;

let person = Person { name: "Alice", age: 30 };
print(person.name);
```

Records can also be generic. See [Generics](./generics.md) for details:

```sage
record Pair<A, B> {
    first: A,
    second: B,
}

let pair = Pair { first: 42, second: "hello" };
```

### Enums

Define types with a fixed set of variants:

```sage
enum Status {
    Active,
    Inactive,
    Pending,
}

enum Direction {
    North,
    South,
    East,
    West,
}
```

Use enum variants directly:

```sage
let s = Active;
let d = North;
```

### Enum Payloads

Enums can carry data:

```sage
enum Result {
    Ok(Int),
    Err(String),
}

enum Message {
    Text(String),
    Number(Int),
    Pair(Int, String),
}

// Construct variants with payloads
let success = Result::Ok(42);
let failure = Result::Err("not found");
let msg = Message::Pair(1, "hello");
```

Enums can also be generic. See [Generics](./generics.md) for details:

```sage
enum Either<L, R> {
    Left(L),
    Right(R),
}

let e = Either::<String, Int>::Left("error");
```

### Match Expressions

Pattern match on enums and other values:

```sage
fn describe(s: Status) -> String {
    return match s {
        Active => "running",
        Inactive => "stopped",
        Pending => "waiting",
    };
}
```

Match on integers with a wildcard:

```sage
fn classify(n: Int) -> String {
    return match n {
        0 => "zero",
        1 => "one",
        _ => "many",
    };
}
```

### Pattern Matching with Payloads

Bind payload values in match arms:

```sage
fn unwrap_result(r: Result) -> String {
    return match r {
        Ok(value) => str(value),
        Err(msg) => msg,
    };
}

fn handle_message(m: Message) -> String {
    return match m {
        Text(s) => s,
        Number(n) => str(n),
        Pair(n, s) => str(n) ++ ": " ++ s,
    };
}
```

The compiler checks that all variants are covered (exhaustiveness checking).

### Constants

Define compile-time constants:

```sage
const MAX_RETRIES: Int = 3;
const DEFAULT_NAME: String = "anonymous";
```

## Agent Types

### Agent\<T\>

A handle to a spawned agent that will emit a value of type `T`:

```sage
agent Worker {
    on start {
        emit(42);
    }
}

agent Main {
    on start {
        let w: Agent<Int> = spawn Worker {};
        let result: Int = try await w;
        emit(result);
    }

    on error(e) {
        emit(0);
    }
}

run Main;
```

### Inferred\<T\>

The result of an LLM inference call:

```sage
let summary = try infer("Summarize: {topic}");
```

`Inferred<T>` can be used anywhere `T` is expected — the type coerces automatically.

## Type Inference

Sage infers types when possible:

```sage
let x = 42;              // Int
let name = "Sage";       // String
let list = [1, 2, 3];    // List<Int>
```

Explicit annotations are required for:
- Function parameters
- Agent state fields
- Closure parameters
- Ambiguous cases

## Type Annotations

Use `: Type` syntax:

```sage
let x: Int = 42;
let items: List<String> = [];

fn double(n: Int) -> Int {
    return n * 2;
}

agent Worker {
    count: Int

    on start {
        emit(self.count * 2);
    }
}
```
