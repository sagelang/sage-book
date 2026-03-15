# Functions

Functions in Sage are defined at the top level and can be called from anywhere.

## Defining Functions

```sage
fn greet(name: String) -> String {
    return "Hello, " ++ name ++ "!";
}

fn add(a: Int, b: Int) -> Int {
    return a + b;
}
```

## Calling Functions

```sage
let message = greet("World");
let sum = add(1, 2);
```

## Return Types

All functions must declare their return type:

```sage
fn double(n: Int) -> Int {
    return n * 2;
}

fn print_message(msg: String) -> Unit {
    print(msg);
    return;
}
```

Use `Unit` for functions that don't return a meaningful value.

## Recursion

Functions can call themselves:

```sage
fn factorial(n: Int) -> Int {
    if n <= 1 {
        return 1;
    }
    return n * factorial(n - 1);
}

fn fibonacci(n: Int) -> Int {
    if n <= 1 {
        return n;
    }
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

## Closures

Sage supports first-class functions and closures:

```sage
// Closure with typed parameters
let add = |x: Int, y: Int| x + y;

// Empty parameter closure
let get_value = || 42;

// Multi-statement closure with block
let greet = |name: String| {
    let msg = "Hello, " ++ name ++ "!";
    return msg;
};
```

Closure parameters require explicit type annotations.

### Function Types

Use `Fn(A, B) -> C` to describe function types:

```sage
fn apply(f: Fn(Int) -> Int, x: Int) -> Int {
    return f(x);
}

let double = |x: Int| x * 2;
let result = apply(double, 21);  // 42
```

### Higher-Order Functions

Functions can return closures:

```sage
fn make_multiplier(n: Int) -> Fn(Int) -> Int {
    return |x: Int| x * n;
}

let triple = make_multiplier(3);
let result = triple(10);  // 30
```

## Fallible Functions

Functions that can fail are marked with `fails`:

```sage
fn risky_operation() -> Int fails {
    let value = try infer("Give me a number");
    return parse_int(value);
}
```

Callers must handle errors with `try` or `catch`:

```sage
agent Main {
    on start {
        let result = try risky_operation();
        emit(result);
    }

    on error(e) {
        emit(0);
    }
}

run Main;
```

## Built-in Functions

Sage provides several built-in functions:

| Function | Signature | Description |
|----------|-----------|-------------|
| `print` | `(String) -> Unit` | Print to console |
| `str` | `(T) -> String` | Convert any value to string |
| `len` | `(List<T>) -> Int` | Get list or map length |
| `push` | `(List<T>, T) -> List<T>` | Append to list |
| `join` | `(List<String>, String) -> String` | Join strings |
| `int_to_str` | `(Int) -> String` | Convert int to string |
| `str_contains` | `(String, String) -> Bool` | Check substring |
| `sleep_ms` | `(Int) -> Unit` | Sleep for milliseconds |
| `map_get` | `(Map<K,V>, K) -> Option<V>` | Get value from map |
| `map_set` | `(Map<K,V>, K, V) -> Unit` | Set key-value in map |
| `map_has` | `(Map<K,V>, K) -> Bool` | Check if key exists |
| `map_delete` | `(Map<K,V>, K) -> Unit` | Remove key from map |
| `map_keys` | `(Map<K,V>) -> List<K>` | Get all keys as list |
| `map_values` | `(Map<K,V>) -> List<V>` | Get all values as list |

## Example

```sage
fn summarize_list(items: List<String>) -> String {
    let count = len(items);
    let joined = join(items, ", ");
    return "Found " ++ str(count) ++ " items: " ++ joined;
}

agent Main {
    on start {
        let result = summarize_list(["apple", "banana", "cherry"]);
        print(result);
        emit(0);
    }
}

run Main;
```

Output:
```
Found 3 items: apple, banana, cherry
```
