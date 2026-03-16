# Assertions

Sage provides a rich set of assertion functions for testing. All assertions are only available in test files (`*_test.sg`).

## Basic Assertions

### assert

Assert that an expression is true:

```sage
test "basic assertion" {
    assert(1 + 1 == 2);
    assert(true);
}
```

### assert_eq / assert_neq

Assert equality or inequality:

```sage
test "equality assertions" {
    assert_eq(1 + 1, 2);
    assert_neq(1 + 1, 3);

    assert_eq("hello", "hello");
    assert_neq("hello", "world");
}
```

### assert_true / assert_false

Assert boolean values:

```sage
test "boolean assertions" {
    assert_true(5 > 3);
    assert_false(5 < 3);
}
```

## Comparison Assertions

### assert_gt / assert_lt

Assert greater than or less than:

```sage
test "comparison assertions" {
    assert_gt(10, 5);   // 10 > 5
    assert_lt(5, 10);   // 5 < 10
}
```

### assert_gte / assert_lte

Assert greater than or equal / less than or equal:

```sage
test "inclusive comparison" {
    assert_gte(10, 10);  // 10 >= 10
    assert_gte(10, 5);   // 10 >= 5
    assert_lte(5, 5);    // 5 <= 5
    assert_lte(5, 10);   // 5 <= 10
}
```

## String Assertions

### assert_contains / assert_not_contains

Assert string containment:

```sage
test "string containment" {
    assert_contains("hello world", "world");
    assert_not_contains("hello world", "foo");
}
```

### assert_starts_with / assert_ends_with

Assert string prefix or suffix:

```sage
test "string prefix and suffix" {
    assert_starts_with("hello world", "hello");
    assert_ends_with("hello world", "world");
}
```

## Collection Assertions

### assert_empty / assert_not_empty

Assert collection emptiness:

```sage
test "collection emptiness" {
    assert_empty([]);
    assert_not_empty([1, 2, 3]);

    assert_empty("");
    assert_not_empty("hello");
}
```

### assert_len

Assert collection length:

```sage
test "collection length" {
    assert_len([1, 2, 3], 3);
    assert_len("hello", 5);
}
```

## Error Assertions

### assert_fails

Assert that an expression produces an error:

```sage
test "agent handles error correctly" {
    mock divine -> fail("simulated failure");

    let handle = summon Summariser { topic: "test" };
    assert_fails(await handle);
}
```

This is useful for testing error handling paths in your agents.

## Assertion Failures

When an assertion fails, the test stops immediately and reports the failure:

```
  FAIL math_test.sg::addition works

Failures:

  math_test.sg::addition works
    thread 'addition_works' panicked at src/main.rs:7:5:
    assertion failed: 1 + 1 == 3
```

The error message shows:
- Which test failed
- Where in the generated code the failure occurred
- The assertion that failed
