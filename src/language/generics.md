# Generics

Sage supports parametric polymorphism (generics), allowing you to write functions, records, and enums that work with any type.

## Generic Functions

### Declaration

Type parameters are declared in angle brackets after the function name:

```sage
fn identity<T>(x: T) -> T {
    return x;
}

fn swap<A, B>(pair: (A, B)) -> (B, A) {
    return (pair.1, pair.0);
}

fn map<T, U>(list: List<T>, f: Fn(T) -> U) -> List<U> {
    let result: List<U> = [];
    for item in list {
        result = push(result, f(item));
    }
    return result;
}
```

Type parameters are typically single uppercase letters (`T`, `U`, `A`, `B`), but any identifier is valid (`Item`, `Key`, `Value`).

### Calling Generic Functions

Type arguments are usually inferred from the arguments:

```sage
let x = identity(42);           // T inferred as Int
let y = identity("hello");      // T inferred as String

let nums = [1, 2, 3];
let doubled = map(nums, |n: Int| n * 2);  // T=Int, U=Int
```

When inference fails or is ambiguous, use turbofish syntax (`::<...>`):

```sage
let empty: List<Int> = [];
let mapped = map::<Int, String>(empty, |n: Int| str(n));
```

### What You Can Do with Type Parameters

Because type parameters are unconstrained, you can only perform operations that work on all types:

**Allowed:**
- Assign values to variables of the same type
- Pass values to other generic functions
- Return values
- Store in generic containers (`List<T>`, `Option<T>`, etc.)
- Use in tuples or record fields

**Not allowed:**
- Use operators like `==`, `+`, `-` on type parameters
- Print type parameters directly (use concrete types)

```sage
// Valid - just moves values around
fn first<T>(list: List<T>) -> Option<T> {
    if len(list) == 0 {
        return None;
    }
    return Some(list[0]);
}

// Invalid - cannot compare unconstrained types
fn contains<T>(list: List<T>, target: T) -> Bool {
    for item in list {
        if item == target {  // Error: cannot apply == to T
            return true;
        }
    }
    return false;
}
```

## Generic Records

### Declaration

Type parameters are declared after the record name:

```sage
record Pair<A, B> {
    first: A,
    second: B,
}

record Page<T> {
    items: List<T>,
    total: Int,
    page: Int,
    page_size: Int,
}

record Timestamped<T> {
    value: T,
    created_at: String,
    updated_at: String,
}
```

### Construction

Type arguments are inferred from field values:

```sage
// Type arguments inferred from field values
let pair = Pair { first: 42, second: "hello" };
// pair: Pair<Int, String>

let page: Page<String> = Page {
    items: ["a", "b", "c"],
    total: 100,
    page: 1,
    page_size: 10,
};
```

### Field Access

Field access works the same as non-generic records:

```sage
let pair = Pair { first: 42, second: "hello" };
let n: Int = pair.first;
let s: String = pair.second;
```

### Generic Records as Parameters

```sage
fn unwrap_timestamped<T>(ts: Timestamped<T>) -> T {
    return ts.value;
}

fn paginate<T>(items: List<T>, page: Int, page_size: Int) -> Page<T> {
    let start = (page - 1) * page_size;
    // ... slice items ...
    return Page {
        items: sliced_items,
        total: len(items),
        page: page,
        page_size: page_size,
    };
}
```

## Generic Enums

### Declaration

Type parameters are declared after the enum name:

```sage
enum Either<L, R> {
    Left(L),
    Right(R),
}

enum Tree<T> {
    Leaf(T),
    Node(Tree<T>, Tree<T>),
}

enum Loadable<T, E> {
    Loading,
    Loaded(T),
    Failed(E),
}
```

### Construction

When constructing a variant, if the type cannot be fully inferred, use turbofish:

```sage
// Type can be inferred from context
let e: Either<String, Int> = Either::Left("error");

// Explicit turbofish when inference fails
let e = Either::<String, Int>::Left("error");
let e2 = Either::<String, Int>::Right(42);

// Tree example
let leaf: Tree<Int> = Tree::Leaf(42);
let tree = Tree::<Int>::Node(Tree::Leaf(1), Tree::Leaf(2));
```

### Pattern Matching

Pattern matching works the same as non-generic enums:

```sage
fn tree_sum(tree: Tree<Int>) -> Int {
    return match tree {
        Leaf(n) => n,
        Node(left, right) => tree_sum(left) + tree_sum(right),
    };
}

fn describe_either<L, R>(e: Either<L, R>) -> String {
    return match e {
        Left(_) => "left",
        Right(_) => "right",
    };
}
```

## Type Inference

### How It Works

Sage infers type arguments from usage:

```sage
fn identity<T>(x: T) -> T { return x; }

let y = identity(42);
// Constraint: T = Int (from argument)
// Result: y: Int
```

### Bidirectional Inference

Type information flows from both arguments and expected return type:

```sage
fn first<T>(list: List<T>) -> Option<T> { ... }

// Inference from argument
let x = first([1, 2, 3]);
// List<T> = List<Int> => T = Int
// Result: x: Option<Int>

// Inference from expected type
let y: Option<String> = first([]);
// Option<T> = Option<String> => T = String
```

### When Inference Fails

Use type annotations or turbofish when inference can't determine the type:

```sage
// Empty list - type unknown
let empty: List<Int> = [];  // Annotation required

// Turbofish on function call
let result = parse::<Int>(json_string);
```

## Using with Built-in Types

The built-in generic types (`List<T>`, `Option<T>`, `Map<K, V>`, `Result<T, E>`) work seamlessly with user-defined generics:

```sage
fn process<T>(items: List<T>) -> Int {
    return len(items);
}

record MyData { value: Int }

let my_items: List<MyData> = [MyData { value: 1 }];
let count = process(my_items);  // T = MyData
```

## Generic Agents

Generic functions can be called from agent handlers:

```sage
fn transform_all<T>(items: List<T>, f: Fn(T) -> T) -> List<T> {
    return map(items, f);
}

agent Processor {
    on start {
        let nums = [1, 2, 3];
        let result = transform_all(nums, |n: Int| n * 2);
        print(str(result));  // [2, 4, 6]
        emit(0);
    }
}

run Processor;
```

## Common Patterns

### Wrapper Types

```sage
record Validated<T> {
    value: T,
    is_valid: Bool,
    errors: List<String>,
}

fn validate<T>(value: T, validator: Fn(T) -> List<String>) -> Validated<T> {
    let errors = validator(value);
    return Validated {
        value: value,
        is_valid: len(errors) == 0,
        errors: errors,
    };
}
```

### Either for Error Handling

```sage
enum Either<L, R> {
    Left(L),
    Right(R),
}

fn safe_divide(a: Int, b: Int) -> Either<String, Int> {
    if b == 0 {
        return Either::<String, Int>::Left("division by zero");
    }
    return Either::<String, Int>::Right(a / b);
}
```

### Pair and Triple

```sage
record Pair<A, B> {
    first: A,
    second: B,
}

fn zip_with_index<T>(items: List<T>) -> List<Pair<Int, T>> {
    let result: List<Pair<Int, T>> = [];
    let i = 0;
    for item in items {
        result = push(result, Pair { first: i, second: item });
        i = i + 1;
    }
    return result;
}
```

## Summary

| Feature | Syntax | Example |
|---------|--------|---------|
| Generic function | `fn name<T>(...)` | `fn identity<T>(x: T) -> T` |
| Generic record | `record Name<T> {...}` | `record Box<T> { value: T }` |
| Generic enum | `enum Name<T> {...}` | `enum Option<T> { Some(T), None }` |
| Turbofish (function) | `name::<Type>(...)` | `parse::<Int>(str)` |
| Turbofish (enum) | `Enum::<Type>::Variant(...)` | `Either::<A, B>::Left(x)` |
| Type annotation | `let x: Type<T> = ...` | `let list: List<Int> = []` |
