# Standard Library Reference

All standard library functions are available in the prelude without import.

## String Functions

### Construction

#### `str(value) -> String`
Convert any value to its string representation.

```sage
str(42)        // "42"
str(true)      // "true"
str([1, 2, 3]) // "[1, 2, 3]"
```

#### `repeat(s, n) -> String`
Repeat a string `n` times.

```sage
repeat("ab", 3)  // "ababab"
repeat("-", 10)  // "----------"
```

### Inspection

#### `len(s) -> Int`
Get the length of a string in characters (Unicode-aware).

```sage
len("hello")  // 5
len("héllo")  // 5 (not bytes!)
len("")       // 0
```

#### `is_empty(s) -> Bool`
Check if a string is empty.

```sage
is_empty("")      // true
is_empty("hello") // false
```

#### `contains(s, sub) -> Bool`
Check if a string contains a substring.

```sage
contains("hello world", "world")  // true
contains("hello", "xyz")          // false
```

#### `starts_with(s, prefix) -> Bool`
Check if a string starts with a prefix.

```sage
starts_with("hello", "hel")  // true
starts_with("hello", "world") // false
```

#### `ends_with(s, suffix) -> Bool`
Check if a string ends with a suffix.

```sage
ends_with("hello.txt", ".txt")  // true
ends_with("hello", "world")     // false
```

#### `index_of(s, sub) -> Option<Int>`
Find the index of a substring. Returns `None` if not found.

```sage
index_of("hello", "ll")   // Some(2)
index_of("hello", "xyz")  // None
```

### Transformation

#### `trim(s) -> String`
Remove whitespace from both ends.

```sage
trim("  hello  ")  // "hello"
trim("\n\thi\n")   // "hi"
```

#### `trim_start(s) -> String`
Remove whitespace from the start.

```sage
trim_start("  hello")  // "hello"
```

#### `trim_end(s) -> String`
Remove whitespace from the end.

```sage
trim_end("hello  ")  // "hello"
```

#### `to_upper(s) -> String`
Convert to uppercase.

```sage
to_upper("hello")  // "HELLO"
```

#### `to_lower(s) -> String`
Convert to lowercase.

```sage
to_lower("HELLO")  // "hello"
```

#### `replace(s, from, to) -> String`
Replace all occurrences of a substring.

```sage
replace("hello world", "world", "sage")  // "hello sage"
replace("aaa", "a", "b")                 // "bbb"
```

#### `replace_first(s, from, to) -> String`
Replace the first occurrence of a substring.

```sage
replace_first("aaa", "a", "b")  // "baa"
```

### Splitting and Joining

#### `split(s, delim) -> List<String>`
Split a string by a delimiter.

```sage
split("a,b,c", ",")     // ["a", "b", "c"]
split("hello", "")      // ["h", "e", "l", "l", "o"]
```

#### `lines(s) -> List<String>`
Split a string into lines.

```sage
lines("a\nb\nc")  // ["a", "b", "c"]
```

#### `join(parts, sep) -> String`
Join strings with a separator.

```sage
join(["a", "b", "c"], ", ")  // "a, b, c"
join(["hello"], "-")         // "hello"
```

### Slicing

#### `slice(s, start, end) -> String`
Extract a substring by character indices (Unicode-aware).

```sage
slice("hello", 1, 4)   // "ell"
slice("héllo", 0, 3)   // "hél"
```

#### `chars(s) -> List<String>`
Split a string into individual characters.

```sage
chars("hello")  // ["h", "e", "l", "l", "o"]
```

### Parsing

#### `parse_int(s) -> Int fails`
Parse a string as an integer.

```sage
let n = try parse_int("42");   // 42
let n = try parse_int("-10");  // -10
let n = try parse_int("abc");  // Error!
```

#### `parse_float(s) -> Float fails`
Parse a string as a float.

```sage
let f = try parse_float("3.14");  // 3.14
let f = try parse_float("42");    // 42.0
```

#### `parse_bool(s) -> Bool fails`
Parse a string as a boolean.

```sage
let b = try parse_bool("true");   // true
let b = try parse_bool("false");  // false
```

---

## List Functions

### Construction

#### `range(start, end) -> List<Int>`
Create a list of integers from start (inclusive) to end (exclusive).

```sage
range(0, 5)   // [0, 1, 2, 3, 4]
range(1, 4)   // [1, 2, 3]
```

#### `range_step(start, end, step) -> List<Int>`
Create a list with a custom step.

```sage
range_step(0, 10, 2)  // [0, 2, 4, 6, 8]
range_step(10, 0, -2) // [10, 8, 6, 4, 2]
```

### Inspection

#### `len(list) -> Int`
Get the length of a list.

```sage
len([1, 2, 3])  // 3
len([])         // 0
```

#### `is_empty(list) -> Bool`
Check if a list is empty.

```sage
is_empty([])       // true
is_empty([1, 2])   // false
```

#### `contains(list, value) -> Bool`
Check if a list contains a value.

```sage
contains([1, 2, 3], 2)  // true
contains([1, 2, 3], 5)  // false
```

#### `first(list) -> Option<T>`
Get the first element.

```sage
first([1, 2, 3])  // Some(1)
first([])         // None
```

#### `last(list) -> Option<T>`
Get the last element.

```sage
last([1, 2, 3])  // Some(3)
last([])         // None
```

#### `get(list, index) -> Option<T>`
Get an element by index.

```sage
get([1, 2, 3], 1)   // Some(2)
get([1, 2, 3], 10)  // None
```

### Transformation

#### `map(list, f) -> List<U>`
Transform each element.

```sage
map([1, 2, 3], |x: Int| x * 2)  // [2, 4, 6]
```

#### `filter(list, f) -> List<T>`
Keep elements that satisfy a predicate.

```sage
filter([1, 2, 3, 4], |x: Int| x > 2)  // [3, 4]
```

#### `reduce(list, init, f) -> U`
Reduce a list to a single value.

```sage
reduce([1, 2, 3], 0, |acc: Int, x: Int| acc + x)  // 6
```

#### `flat_map(list, f) -> List<U>`
Map and flatten.

```sage
flat_map([1, 2], |x: Int| [x, x * 10])  // [1, 10, 2, 20]
```

#### `flatten(list) -> List<T>`
Flatten a list of lists.

```sage
flatten([[1, 2], [3, 4]])  // [1, 2, 3, 4]
```

### Ordering

#### `sort(list) -> List<T>`
Sort a list in ascending order.

```sage
sort([3, 1, 2])  // [1, 2, 3]
```

#### `reverse(list) -> List<T>`
Reverse a list.

```sage
reverse([1, 2, 3])  // [3, 2, 1]
```

### Slicing

#### `slice(list, start, end) -> List<T>`
Extract a sublist.

```sage
slice([1, 2, 3, 4, 5], 1, 4)  // [2, 3, 4]
```

#### `take(list, n) -> List<T>`
Take the first n elements.

```sage
take([1, 2, 3, 4], 2)  // [1, 2]
```

#### `drop(list, n) -> List<T>`
Drop the first n elements.

```sage
drop([1, 2, 3, 4], 2)  // [3, 4]
```

### Aggregation

#### `any(list, f) -> Bool`
Check if any element satisfies a predicate.

```sage
any([1, 2, 3], |x: Int| x > 2)  // true
```

#### `all(list, f) -> Bool`
Check if all elements satisfy a predicate.

```sage
all([1, 2, 3], |x: Int| x > 0)  // true
```

#### `count(list, f) -> Int`
Count elements satisfying a predicate.

```sage
count([1, 2, 3, 4], |x: Int| x > 2)  // 2
```

#### `sum(list) -> Int`
Sum integers.

```sage
sum([1, 2, 3])  // 6
```

#### `sum_float(list) -> Float`
Sum floats.

```sage
sum_float([1.5, 2.5])  // 4.0
```

### Mutation Helpers

#### `push(list, value) -> List<T>`
Add an element to the end (returns new list).

```sage
push([1, 2], 3)  // [1, 2, 3]
```

#### `concat(a, b) -> List<T>`
Concatenate two lists.

```sage
concat([1, 2], [3, 4])  // [1, 2, 3, 4]
```

#### `unique(list) -> List<T>`
Remove duplicates.

```sage
unique([1, 2, 2, 3, 1])  // [1, 2, 3]
```

#### `zip(a, b) -> List<(T, U)>`
Combine two lists into pairs.

```sage
zip([1, 2], ["a", "b"])  // [(1, "a"), (2, "b")]
```

#### `enumerate(list) -> List<(Int, T)>`
Pair each element with its index.

```sage
enumerate(["a", "b"])  // [(0, "a"), (1, "b")]
```

---

## Math Functions

### Basic

#### `abs(n) -> Int`
Absolute value of an integer.

```sage
abs(-5)  // 5
abs(5)   // 5
```

#### `abs_float(n) -> Float`
Absolute value of a float.

```sage
abs_float(-3.14)  // 3.14
```

#### `min(a, b) -> Int`
Minimum of two integers.

```sage
min(3, 7)  // 3
```

#### `max(a, b) -> Int`
Maximum of two integers.

```sage
max(3, 7)  // 7
```

#### `min_float(a, b) -> Float`
Minimum of two floats.

#### `max_float(a, b) -> Float`
Maximum of two floats.

#### `clamp(value, low, high) -> Int`
Clamp a value to a range.

```sage
clamp(5, 0, 10)   // 5
clamp(-5, 0, 10)  // 0
clamp(15, 0, 10)  // 10
```

### Rounding

#### `floor(n) -> Int`
Round down to nearest integer.

```sage
floor(3.7)   // 3
floor(-3.7)  // -4
```

#### `ceil(n) -> Int`
Round up to nearest integer.

```sage
ceil(3.2)   // 4
ceil(-3.2)  // -3
```

#### `round(n) -> Int`
Round to nearest integer.

```sage
round(3.5)  // 4
round(3.4)  // 3
```

### Powers and Roots

#### `pow(base, exp) -> Int`
Integer power.

```sage
pow(2, 10)  // 1024
pow(3, 3)   // 27
```

#### `pow_float(base, exp) -> Float`
Float power.

```sage
pow_float(2.0, 0.5)  // 1.414...
```

#### `sqrt(n) -> Float`
Square root.

```sage
sqrt(16.0)  // 4.0
sqrt(2.0)   // 1.414...
```

#### `log(n) -> Float`
Natural logarithm.

```sage
log(E)  // 1.0
```

#### `log2(n) -> Float`
Base-2 logarithm.

```sage
log2(8.0)  // 3.0
```

#### `log10(n) -> Float`
Base-10 logarithm.

```sage
log10(100.0)  // 2.0
```

### Conversion

#### `int_to_float(n) -> Float`
Convert integer to float.

```sage
int_to_float(42)  // 42.0
```

#### `float_to_int(n) -> Int`
Convert float to integer (truncates).

```sage
float_to_int(3.9)  // 3
```

### Constants

```sage
const PI: Float = 3.141592653589793
const E: Float = 2.718281828459045
```

---

## I/O Functions

### File Operations

#### `read_file(path) -> String fails`
Read entire file contents.

```sage
let contents = try read_file("data.txt");
```

#### `write_file(path, content) fails`
Write string to file (creates or truncates).

```sage
try write_file("output.txt", "Hello, world!");
```

#### `append_file(path, content) fails`
Append string to file.

```sage
try append_file("log.txt", "New entry\n");
```

#### `file_exists(path) -> Bool`
Check if a file or directory exists.

```sage
if file_exists("config.json") {
    // ...
}
```

#### `delete_file(path) fails`
Delete a file.

```sage
try delete_file("temp.txt");
```

#### `list_dir(path) -> List<String> fails`
List directory contents.

```sage
let files = try list_dir(".");
```

#### `make_dir(path) fails`
Create a directory (and parents).

```sage
try make_dir("output/data");
```

### Standard Streams

#### `read_line() -> String fails`
Read a line from stdin.

```sage
print("Enter your name: ");
let name = try read_line();
```

#### `read_all() -> String fails`
Read all input from stdin until EOF.

```sage
let input = try read_all();
```

---

## Time Functions

#### `now_ms() -> Int`
Current time in milliseconds since Unix epoch.

```sage
let timestamp = now_ms();
```

#### `now_s() -> Int`
Current time in seconds since Unix epoch.

```sage
let timestamp = now_s();
```

#### `format_timestamp(ms, fmt) -> String`
Format a timestamp.

```sage
format_timestamp(now_ms(), "%Y-%m-%d")  // "2024-01-15"
format_timestamp(now_ms(), "%H:%M:%S")  // "10:30:45"
```

Format codes:
- `%Y` — year (4 digits)
- `%m` — month (01-12)
- `%d` — day (01-31)
- `%H` — hour (00-23)
- `%M` — minute (00-59)
- `%S` — second (00-59)
- `%F` — ISO date (YYYY-MM-DD)
- `%T` — ISO time (HH:MM:SS)

#### `parse_timestamp(s, fmt) -> Int fails`
Parse a timestamp string.

```sage
let ms = try parse_timestamp("2024-01-15 10:30:00 +0000", "%Y-%m-%d %H:%M:%S %z");
```

### Constants

```sage
const MS_PER_SECOND: Int = 1000
const MS_PER_MINUTE: Int = 60000
const MS_PER_HOUR: Int = 3600000
const MS_PER_DAY: Int = 86400000
```

---

## Option Functions

#### `is_some(opt) -> Bool`
Check if option has a value.

```sage
is_some(Some(42))  // true
is_some(None)      // false
```

#### `is_none(opt) -> Bool`
Check if option is empty.

```sage
is_none(None)      // true
is_none(Some(42))  // false
```

#### `unwrap(opt) -> T fails`
Extract value or fail.

```sage
let x = try unwrap(Some(42));  // 42
let y = try unwrap(None);      // Error!
```

#### `unwrap_or(opt, default) -> T`
Extract value or return default.

```sage
unwrap_or(Some(42), 0)  // 42
unwrap_or(None, 0)      // 0
```

#### `unwrap_or_else(opt, f) -> T`
Extract value or compute default.

```sage
unwrap_or_else(None, || expensive_default())
```

#### `map_option(opt, f) -> Option<U>`
Transform the value if present.

```sage
map_option(Some(2), |x: Int| x * 2)  // Some(4)
map_option(None, |x: Int| x * 2)     // None
```

---

## JSON Functions

#### `json_parse(s) -> String fails`
Validate JSON and return if valid.

```sage
let json = try json_parse("{\"name\": \"Alice\"}");
```

#### `json_get(json, key) -> Option<String>`
Get a field as a string.

```sage
json_get("{\"name\": \"Alice\"}", "name")  // Some("Alice")
json_get("{\"age\": 30}", "name")          // None
```

#### `json_get_int(json, key) -> Option<Int>`
Get a field as an integer.

```sage
json_get_int("{\"age\": 30}", "age")  // Some(30)
```

#### `json_get_float(json, key) -> Option<Float>`
Get a field as a float.

```sage
json_get_float("{\"price\": 9.99}", "price")  // Some(9.99)
```

#### `json_get_bool(json, key) -> Option<Bool>`
Get a field as a boolean.

```sage
json_get_bool("{\"active\": true}", "active")  // Some(true)
```

#### `json_get_list(json, key) -> Option<List<String>>`
Get a field as a list of strings.

```sage
json_get_list("{\"tags\": [\"a\", \"b\"]}", "tags")  // Some(["a", "b"])
```

#### `json_stringify(value) -> String`
Convert a value to JSON string.

```sage
json_stringify("hello")  // "\"hello\""
```

---

## Map Functions

#### `map_get(map, key) -> Option<V>`
Get a value by key.

```sage
let ages = {"alice": 30, "bob": 25};
map_get(ages, "alice")  // Some(30)
map_get(ages, "charlie") // None
```

#### `map_set(map, key, value)`
Set a key-value pair (mutates map).

```sage
let ages = {"alice": 30};
map_set(ages, "bob", 25);
```

#### `map_has(map, key) -> Bool`
Check if key exists.

```sage
map_has({"a": 1}, "a")  // true
map_has({"a": 1}, "b")  // false
```

#### `map_delete(map, key)`
Remove a key (mutates map).

```sage
let m = {"a": 1, "b": 2};
map_delete(m, "a");
```

#### `map_keys(map) -> List<K>`
Get all keys.

```sage
map_keys({"a": 1, "b": 2})  // ["a", "b"]
```

#### `map_values(map) -> List<V>`
Get all values.

```sage
map_values({"a": 1, "b": 2})  // [1, 2]
```

---

## Output

#### `print(message)`
Print to stdout with newline.

```sage
print("Hello, world!");
print("Value: " ++ str(42));
```
