---
source_course: "rust"
source_lesson: "rust-enums-match"
---

# Enums: Powerful Algebraic Data Types

Unlike C-style enums, Rust enums can hold data.

```rust
enum Message {
    Quit,                       // No data
    Move { x: i32, y: i32 },    // Named fields (like struct)
    Write(String),              // Single value
    ChangeColor(i32, i32, i32), // Multiple values (like tuple)
}
```

## The Option Enum

Rust has no null. Instead, use `Option<T>`:

```rust
enum Option<T> {
    Some(T),
    None,
}

let some_number = Some(5);
let no_number: Option<i32> = None;
```

## The `match` Control Flow

`match` allows you to compare a value against patterns. It must be **exhaustive**.

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}", state);
            25
        }
    }
}
```

## Match Guards & Binding

```rust
match num {
    n if n < 0 => println!("Negative"),
    n @ 1..=10 => println!("Between 1-10: {n}"),
    n => println!("Other: {n}"),
}
```

## The `_` Placeholder

```rust
match value {
    1 => println!("One"),
    _ => (), // Catch-all, do nothing
}
```

See [Enums and Pattern Matching](https://doc.rust-lang.org/stable/book/ch06-00-enums.html).

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