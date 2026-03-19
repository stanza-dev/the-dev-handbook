---
source_course: "rust-functional"
source_lesson: "rust-func-question-mark-patterns"
---

# Advanced ? Operator Patterns

## Introduction
The `?` operator goes beyond simple error propagation. It works with both `Result` and `Option`, supports automatic error conversion via `From`/`Into`, and enables clean patterns for combining different fallible operations. This lesson covers advanced usage that makes Rust error handling ergonomic.

## Key Concepts
- **`?` on Option**: Returns `None` early from functions that return `Option<T>`. Useful for chaining lookups.
- **Automatic `From` conversion**: `?` calls `.into()` on the error, converting it to the function's return error type if a `From` impl exists.
- **`?` in main**: Since Rust 1.26, `main()` can return `Result<(), E>`, enabling `?` in the program entry point.

## Real World Context
Production Rust code uses `?` extensively. Web frameworks like Actix and Axum rely on automatic error conversion so handlers can use `?` with different error types. Understanding how `?` interacts with `From` is essential for designing clean error hierarchies.

## Deep Dive

### ? with Option in method chains

The `?` operator works inside functions that return `Option`:

```rust
fn get_first_word_length(text: Option<&str>) -> Option<usize> {
    let text = text?;  // Returns None if text is None
    let first_word = text.split_whitespace().next()?;  // Returns None if empty
    Some(first_word.len())
}

assert_eq!(get_first_word_length(Some("hello world")), Some(5));
assert_eq!(get_first_word_length(Some("")), None);
assert_eq!(get_first_word_length(None), None);
```

Each `?` short-circuits to `None` if the value is absent, keeping the code flat.

### Automatic error conversion with From

When your function returns `Result<T, MyError>` and you use `?` on a `Result<T, OtherError>`, Rust calls `MyError::from(other_error)` automatically:

```rust
#[derive(Debug)]
enum ConfigError {
    Io(std::io::Error),
    Parse(serde_json::Error),
}

impl From<std::io::Error> for ConfigError {
    fn from(e: std::io::Error) -> Self {
        ConfigError::Io(e)
    }
}

impl From<serde_json::Error> for ConfigError {
    fn from(e: serde_json::Error) -> Self {
        ConfigError::Parse(e)
    }
}

fn load_config(path: &str) -> Result<Config, ConfigError> {
    let content = std::fs::read_to_string(path)?;  // io::Error -> ConfigError::Io
    let config = serde_json::from_str(&content)?;   // serde::Error -> ConfigError::Parse
    Ok(config)
}
```

The `From` impls let `?` convert errors seamlessly. No `map_err` needed.

### ? in main()

You can return `Result` from `main` to use `?` at the top level:

```rust
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let config = load_config("settings.json")?;
    let db = connect_database(&config.db_url)?;
    start_server(db)?;
    Ok(())
}
```

If any step fails, the program prints the error and exits with a non-zero status code.

### Converting Option to Result with ?

Combine `ok_or` with `?` to use Options in Result-returning functions:

```rust
fn get_user_email(users: &HashMap<u32, User>, id: u32) -> Result<String, AppError> {
    let user = users.get(&id)
        .ok_or(AppError::NotFound(id))?;
    let email = user.email.as_ref()
        .ok_or(AppError::MissingEmail(id))?;
    Ok(email.clone())
}
```

Each `ok_or` converts a `None` into a meaningful error, and `?` propagates it.

## Common Pitfalls
1. **Using `?` in closures** — The `?` operator returns from the enclosing *function*, not the closure. Inside a closure passed to `map`, use combinators instead.
2. **Missing `From` implementations** — If `?` gives a "cannot convert" error, you need a `From<SourceError> for TargetError` implementation.
3. **Mixing `?` on Option and Result** — You cannot use `?` on an `Option` inside a function that returns `Result` (or vice versa) without conversion. Use `.ok_or()` to bridge.

## Best Practices
1. **Implement `From` for your error enum** — This unlocks seamless `?` usage across different error types.
2. **Use `Box<dyn Error>` for prototyping** — It accepts any error type, letting you use `?` freely. Refine to a custom enum later.
3. **Keep `?` chains short** — If a function has many `?` calls, consider splitting it into smaller functions for readability.

## Summary
- `?` works on both `Result` and `Option`.
- Automatic `From` conversion lets `?` handle error type differences.
- `main()` can return `Result` for top-level error handling.
- Use `ok_or` to bridge `Option` into `Result` for `?` usage.
- `?` does not work inside closures — use combinators there instead.

## Code Examples

**Converting nested HashMap lookups (Option) into Result with meaningful errors using ok_or_else and ?**

```rust
use std::collections::HashMap;

#[derive(Debug)]
enum LookupError {
    NotFound(String),
    InvalidFormat(String),
}

// Chaining ? with Option-to-Result conversion
fn resolve_nested_value(
    data: &HashMap<String, HashMap<String, String>>,
    section: &str,
    key: &str,
) -> Result<i32, LookupError> {
    let inner = data.get(section)
        .ok_or_else(|| LookupError::NotFound(section.into()))?;

    let raw_value = inner.get(key)
        .ok_or_else(|| LookupError::NotFound(key.into()))?;

    raw_value.parse::<i32>()
        .map_err(|_| LookupError::InvalidFormat(raw_value.clone()))
}

// Usage:
// let val = resolve_nested_value(&config, "database", "port")?;
```


## Resources

- [Recoverable Errors with Result](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html) — Rust Book chapter covering Result, the ? operator, and error propagation patterns

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*