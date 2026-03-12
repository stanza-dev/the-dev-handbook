---
source_course: "rust"
source_lesson: "rust-enums-match"
---

# Enums & Pattern Matching

## Introduction

Rust enums are far more powerful than enums in most other languages. Each variant can hold different types and amounts of data, making them algebraic data types. Combined with `match`, they give you exhaustive, type-safe control flow that the compiler verifies at compile time.

## Key Concepts

- **Enum**: A type that can be one of several variants. Each variant can optionally hold data (no data, named fields, unnamed fields, or a single value).
- **`Option<T>`**: Rust's replacement for null. A value is either `Some(T)` or `None`, forcing you to handle the absent case.
- **`match` expression**: A control flow construct that compares a value against patterns and executes the matching arm. It must be exhaustive — every possible variant must be handled.
- **Match guard**: An additional `if` condition on a match arm, like `n if n < 0 => ...`.

## Real World Context

Enums model state machines, message types, command patterns, and error variants in production Rust. The `Result<T, E>` enum powers Rust's entire error handling system. HTTP libraries use enums for request methods (`GET`, `POST`, `PUT`). Any time your data has a finite set of possible shapes, an enum is the right tool.

## Deep Dive

Rust enum variants can hold different data types:

```rust
enum Message {
    Quit,                       // No data (unit variant)
    Move { x: i32, y: i32 },    // Named fields (struct variant)
    Write(String),              // Single value (tuple variant)
    ChangeColor(i32, i32, i32), // Multiple values (tuple variant)
}
```

The `Option<T>` enum replaces null pointers:

```rust
let some_number: Option<i32> = Some(5);
let no_number: Option<i32> = None;
```

You must handle both cases before using the inner value, which prevents null pointer exceptions entirely.

The `match` expression destructures enum variants and must cover all possibilities:

```rust
fn describe(msg: Message) {
    match msg {
        Message::Quit => println!("Quitting"),
        Message::Move { x, y } => println!("Moving to ({x}, {y})"),
        Message::Write(text) => println!("Writing: {text}"),
        Message::ChangeColor(r, g, b) => println!("Color: ({r},{g},{b})"),
    }
}
```

Match guards add conditional logic to arms:

```rust
match num {
    n if n < 0 => println!("Negative"),
    0 => println!("Zero"),
    n => println!("Positive: {n}"),
}
```

The `_` wildcard catches any remaining cases:

```rust
match value {
    1 => println!("One"),
    _ => println!("Something else"),
}
```

## Common Pitfalls

1. **Forgetting to handle all variants** — `match` must be exhaustive. If you add a new variant to an enum, every `match` on that enum must be updated. Use `_` as a catch-all only when you genuinely want to ignore remaining variants.
2. **Trying to use `Option<T>` values directly** — You cannot add an `Option<i32>` to an `i32`. You must unwrap or match the `Option` first. This is by design: it forces you to handle the `None` case.

## Best Practices

1. **Model your domain with enums** — When a value can be one of several shapes, use an enum instead of boolean flags or string tags. The compiler will enforce exhaustiveness.
2. **Prefer `match` over chains of `if/else`** — `match` is more readable and the compiler verifies you have covered all variants.

## Summary

- Rust enums can hold data in each variant, making them algebraic data types.
- `Option<T>` replaces null: values are either `Some(T)` or `None`.
- `match` must be exhaustive — every variant must be handled, or a `_` wildcard used.
- Match guards add conditional logic to pattern arms.
- Enums combined with `match` give you type-safe, compiler-verified control flow.

## Code Examples

**Matching Option**

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);    // Some(6)
let none = plus_one(None);   // None
```


## Resources

- [Enums and Pattern Matching](https://doc.rust-lang.org/stable/book/ch06-00-enums.html) — Complete guide to enums

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*