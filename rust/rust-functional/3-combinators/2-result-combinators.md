---
source_course: "rust-functional"
source_lesson: "rust-func-result-combinators"
---

# Result<T, E> Combinators

Similar to Option, but for fallible operations.

## Transforming Values

```rust
let x: Result<i32, &str> = Ok(5);

// map: Transform Ok value
let doubled = x.map(|n| n * 2);  // Ok(10)

// map_err: Transform Err value
let converted = x.map_err(|e| format!("Error: {e}"));
```

## Chaining Results

```rust
fn parse_number(s: &str) -> Result<i32, ParseError> { ... }
fn validate(n: i32) -> Result<i32, ValidationError> { ... }

// and_then: Chain fallible operations
let result = parse_number("42")
    .and_then(|n| validate(n));

// or_else: Try alternative on error
let result = parse_number("abc")
    .or_else(|_| parse_number("42"));
```

## The ? Operator

```rust
// ? is sugar for and_then with early return
fn process() -> Result<String, Error> {
    let a = step_one()?;      // Early return on Err
    let b = step_two(a)?;
    let c = step_three(b)?;
    Ok(format!("Done: {c}"))
}

// Equivalent to:
fn process() -> Result<String, Error> {
    let a = match step_one() {
        Ok(v) => v,
        Err(e) => return Err(e.into()),
    };
    // ...
}
```

## Collecting Results

```rust
let strings = vec!["1", "2", "3"];

// Collect Vec<Result<T, E>> into Result<Vec<T>, E>
let numbers: Result<Vec<i32>, _> = strings
    .iter()
    .map(|s| s.parse())
    .collect();
// Ok([1, 2, 3]) or first Err
```

## Code Examples

**Result combinator patterns**

```rust
// Error handling with combinators
use std::fs;
use std::io;

fn read_config() -> Result<Config, ConfigError> {
    fs::read_to_string("config.json")
        .map_err(|e| ConfigError::Io(e))
        .and_then(|contents| {
            serde_json::from_str(&contents)
                .map_err(|e| ConfigError::Parse(e))
        })
}

// Combining multiple Results
fn fetch_data() -> Result<(User, Posts), Error> {
    let user = get_user()?;
    let posts = get_posts(user.id)?;
    Ok((user, posts))
}

// unwrap_or_else for Result
let config = read_config()
    .unwrap_or_else(|e| {
        eprintln!("Warning: {e}, using defaults");
        Config::default()
    });

// inspect/inspect_err for debugging
let result = parse_input()
    .inspect(|v| println!("Parsed: {v}"))
    .inspect_err(|e| eprintln!("Error: {e}"))
    .and_then(process);
```


---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*