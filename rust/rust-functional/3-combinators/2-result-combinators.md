---
source_course: "rust-functional"
source_lesson: "rust-func-result-combinators"
---

# Result Combinators

## Introduction
`Result<T, E>` is Rust's primary error handling type, and it supports the same combinator style as `Option`. By chaining `map`, `and_then`, `map_err`, and the `?` operator, you can write error handling code that is both safe and expressive without deeply nested match arms.

## Key Concepts
- **`map`**: Transforms the `Ok` value, passing through `Err` unchanged.
- **`map_err`**: Transforms the `Err` value, passing through `Ok` unchanged. Essential for converting between error types.
- **`and_then`**: Chains operations that return `Result`, flattening `Result<Result<T, E>, E>` into `Result<T, E>`.
- **`?` operator**: Sugar for early return on `Err`. Equivalent to `match result { Ok(v) => v, Err(e) => return Err(e.into()) }`.

## Real World Context
Every I/O operation, parse call, and network request in Rust returns a `Result`. Combinator chains let you build multi-step pipelines — read a file, parse its contents, validate the data — where any step can fail, and the error propagates cleanly to the caller.

## Deep Dive

### Transforming Ok values with map

```rust
let parsed: Result<i32, _> = "42".parse::<i32>();

let doubled: Result<i32, _> = parsed.map(|n| n * 2);
assert_eq!(doubled, Ok(84));
```

`map` only runs the closure on `Ok` values. If `parsed` were `Err`, the error passes through untouched.

### Transforming errors with map_err

When combining functions with different error types, use `map_err` to unify them:

```rust
use std::fs;

fn read_config(path: &str) -> Result<Config, AppError> {
    let contents = fs::read_to_string(path)
        .map_err(|e| AppError::Io(e))?;

    serde_json::from_str(&contents)
        .map_err(|e| AppError::Parse(e))
}
```

`map_err` converts `io::Error` to `AppError::Io` and `serde_json::Error` to `AppError::Parse`, keeping the pipeline clean.

### Chaining with and_then

When each step returns a `Result`, use `and_then` to chain without nesting:

```rust
fn parse_port(input: &str) -> Result<u16, String> {
    input.parse::<u16>()
        .map_err(|e| format!("Invalid port: {e}"))
}

fn validate_port(port: u16) -> Result<u16, String> {
    if port > 1024 {
        Ok(port)
    } else {
        Err(format!("Port {port} is privileged"))
    }
}

let result = parse_port("8080")
    .and_then(validate_port);
assert_eq!(result, Ok(8080));

let invalid = parse_port("80")
    .and_then(validate_port);
assert_eq!(invalid, Err("Port 80 is privileged".to_string()));
```

If `parse_port` fails, `validate_port` is never called.

### The ? operator

The `?` operator is the most common way to propagate errors in Rust:

```rust
fn process_file(path: &str) -> Result<Vec<i32>, Box<dyn std::error::Error>> {
    let content = fs::read_to_string(path)?; // Returns early on Err
    let numbers: Vec<i32> = content
        .lines()
        .map(|line| line.trim().parse::<i32>())
        .collect::<Result<Vec<_>, _>>()?; // Collects Results, returns on first Err
    Ok(numbers)
}
```

Each `?` either unwraps the `Ok` value or returns the error to the caller. The `Into` trait is called on the error, allowing automatic conversion.

### Collecting Results from iterators

You can collect an iterator of `Result` values into a single `Result` containing a collection:

```rust
let inputs = vec!["1", "2", "three", "4"];

let parsed: Result<Vec<i32>, _> = inputs.iter()
    .map(|s| s.parse::<i32>())
    .collect();

// Err — "three" fails to parse, so the whole collection fails
assert!(parsed.is_err());
```

This pattern is extremely useful for batch validation.

## Common Pitfalls
1. **Using `unwrap()` for error handling** — `unwrap()` panics on `Err`. Use `?`, `unwrap_or_else`, or explicit matching instead.
2. **Forgetting `map_err` when error types differ** — If function A returns `io::Error` and function B returns `serde::Error`, you need `map_err` to convert to a common error type.
3. **Using `and_then` when `map` suffices** — If your closure returns a plain value (not a `Result`), use `map`. `and_then` is for closures returning `Result`.

## Best Practices
1. **Prefer `?` for straightforward propagation** — For most functions, `?` is cleaner than explicit combinator chains.
2. **Use `map_err` at API boundaries** — Convert library errors to your domain error type at the point of use.
3. **Combine `inspect` / `inspect_err` for logging** — `result.inspect_err(|e| log::warn!("Failed: {e}"))` logs without consuming the error.

## Summary
- `map` transforms Ok values; `map_err` transforms Err values.
- `and_then` chains fallible operations without nesting.
- The `?` operator is syntactic sugar for early return on Err.
- Collecting `Result` iterators yields `Result<Collection, E>` — fails on the first error.
- Use `inspect_err` for non-destructive error logging.

## Code Examples

**A complete error-handling pipeline showing both combinator style and ? operator style for the same logic**

```rust
use std::fs;
use std::num::ParseIntError;

#[derive(Debug)]
enum AppError {
    Io(std::io::Error),
    Parse(ParseIntError),
    Validation(String),
}

// Full pipeline: read file -> parse lines -> validate -> collect
fn load_scores(path: &str) -> Result<Vec<u32>, AppError> {
    fs::read_to_string(path)
        .map_err(AppError::Io)
        .and_then(|content| {
            content.lines()
                .map(|line| {
                    line.trim()
                        .parse::<u32>()
                        .map_err(AppError::Parse)
                        .and_then(|score| {
                            if score <= 100 {
                                Ok(score)
                            } else {
                                Err(AppError::Validation(
                                    format!("Score {score} exceeds 100")
                                ))
                            }
                        })
                })
                .collect()
        })
}

// Using ? for the same logic — often cleaner
fn load_scores_with_question_mark(path: &str) -> Result<Vec<u32>, AppError> {
    let content = fs::read_to_string(path).map_err(AppError::Io)?;
    content.lines()
        .map(|line| {
            let score: u32 = line.trim().parse().map_err(AppError::Parse)?;
            if score <= 100 {
                Ok(score)
            } else {
                Err(AppError::Validation(format!("Score {score} exceeds 100")))
            }
        })
        .collect()
}
```


## Resources

- [Result enum documentation](https://doc.rust-lang.org/std/result/enum.Result.html) — Complete API reference for Result with all combinator methods and examples

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*