---
source_course: "rust"
source_lesson: "rust-result-error-handling"
---

# Error Handling with Result

## Introduction

Rust does not have exceptions. Instead, it uses the `Result<T, E>` enum for recoverable errors and `panic!` for unrecoverable ones. This design forces you to handle errors explicitly, leading to more robust programs.

## Key Concepts

- **Result<T, E>**: An enum with two variants: `Ok(T)` for success and `Err(E)` for failure. It is returned by any operation that can fail.
- **The `?` operator**: Syntactic sugar that unwraps `Ok` or returns `Err` early from the enclosing function, propagating the error to the caller.
- **unwrap / expect**: Convenience methods that extract the `Ok` value or panic on `Err`. `expect` lets you attach a custom panic message.

## Real World Context

Every production Rust application uses `Result` extensively: reading files, parsing configuration, making network requests, querying databases. The `?` operator makes error propagation concise, and libraries like `anyhow` and `thiserror` build on `Result` to provide ergonomic error handling at scale.

## Deep Dive

The `Result` enum is defined as:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

You handle it with `match`, which forces you to deal with both cases:

```rust
use std::fs::File;

let file = match File::open("config.toml") {
    Ok(f) => f,
    Err(e) => panic!("Cannot open config: {:?}", e),
};
```

For quick prototyping, `unwrap` and `expect` provide shortcuts:

```rust
let file = File::open("config.toml").unwrap();
let file = File::open("config.toml")
    .expect("config.toml must exist in project root");
```

The `?` operator is the idiomatic way to propagate errors. It replaces verbose match arms with a single character:

```rust
use std::io::{self, Read};

fn read_config() -> Result<String, io::Error> {
    let mut content = String::new();
    File::open("config.toml")?.read_to_string(&mut content)?;
    Ok(content)
}
```

Each `?` either unwraps the `Ok` value and continues, or returns the `Err` from the function immediately.

## Common Pitfalls

1. **Using `unwrap` in library code** — `unwrap` panics on error, crashing the program. In libraries, always return `Result` so callers can decide how to handle failures.
2. **Ignoring `Result` warnings** — Rust warns when a `Result` is unused. Never suppress this warning; either handle the error or explicitly ignore it with `let _ = ...`.

## Best Practices

1. **Use `?` for propagation** — It keeps error-handling code concise and readable. Reserve `match` for cases where you need to handle specific error variants differently.
2. **Use `expect` with descriptive messages** — When you do need to unwrap (e.g., in tests or `main`), use `expect("why this should not fail")` so the panic message explains the assumption that was violated.

## Summary

- `Result<T, E>` replaces exceptions with explicit, type-safe error handling.
- Use `match` for granular error handling, `?` for propagation, and `expect` for cases with clear invariants.
- Never use `unwrap` in production library code.
- The `?` operator works in any function that returns `Result` (or `Option`).
- Rust's error model catches unhandled errors at compile time.

## Code Examples

**Error propagation**

```rust
use std::fs;

// Chaining with ?
fn read_username() -> Result<String, std::io::Error> {
    fs::read_to_string("hello.txt")
}

// In main with Result return
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let username = read_username()?;
    println!("Username: {username}");
    Ok(())
}
```


## Resources

- [Error Handling](https://doc.rust-lang.org/stable/book/ch09-00-error-handling.html) — Complete guide to error handling in Rust

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*