# Basic Syntax

Sage syntax is designed to be familiar to developers coming from Rust, TypeScript, or Go.

## Comments

```sage
// Single-line comment

/*
   Multi-line comment
   (not yet supported)
*/
```

## Variables

Variables are declared with `let`:

```sage
let x = 42;
let name = "Sage";
let numbers = [1, 2, 3];
```

Variables are immutable by default. Reassignment creates a new binding:

```sage
let x = 1;
x = 2;  // Reassigns x
```

## Operators

### Arithmetic
```sage
let sum = 1 + 2;
let diff = 5 - 3;
let product = 4 * 2;
let quotient = 10 / 2;
```

### Comparison
```sage
let eq = x == y;
let neq = x != y;
let lt = x < y;
let gt = x > y;
let lte = x <= y;
let gte = x >= y;
```

### Logical
```sage
let and = a && b;
let or = a || b;
let not = !a;
```

### String Concatenation
```sage
let greeting = "Hello, " ++ name ++ "!";
```

## String Interpolation

Strings support interpolation with `{identifier}`:

```sage
let name = "World";
let greeting = "Hello, {name}!";  // "Hello, World!"
```

## Semicolons

Following Rust conventions:
- **Required** after: `let`, `return`, assignments, expression statements, `run`
- **Not required** after: `if`/`else`, `for`, `while` blocks

```sage
let x = 1;           // semicolon required
if x > 0 {           // no semicolon after block
    print("positive");
}
```
