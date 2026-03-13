---
source_course: "rust-performance"
source_lesson: "rust-perf-newtypes"
---

# Newtypes: Type Safety at Zero Cost

## Introduction

The newtype pattern wraps an existing type in a single-field struct, providing type safety and API boundaries with zero runtime overhead. The compiler optimizes the wrapper away completely.

## Key Concepts

### Basic Newtype

```rust
struct UserId(u64);
struct OrderId(u64);

fn get_user(id: UserId) -> User { ... }
fn get_order(id: OrderId) -> Order { ... }

// Compile error: cannot pass OrderId where UserId expected
get_user(OrderId(42)); // Error!
```

Both `UserId` and `OrderId` are `u64` at runtime, but the compiler prevents mixing them up.

### `repr(transparent)`

```rust
#[repr(transparent)]
struct Meters(f64);
```

`repr(transparent)` guarantees the newtype has the exact same memory layout as its inner type. This is essential for FFI and transmutation safety.

## Real World Context

Newtypes are ubiquitous in production Rust: database IDs, validated strings, units of measurement, and API boundary types. They are the foundation of type-driven development.

## Deep Dive

### Implementing Traits for Newtypes

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
struct Celsius(f64);

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
struct Fahrenheit(f64);

impl Celsius {
    fn to_fahrenheit(self) -> Fahrenheit {
        Fahrenheit(self.0 * 9.0 / 5.0 + 32.0)
    }
}

impl From<Celsius> for Fahrenheit {
    fn from(c: Celsius) -> Self {
        c.to_fahrenheit()
    }
}
```

### Validation on Construction

```rust
struct NonEmptyString(String);

impl NonEmptyString {
    fn new(s: String) -> Option<Self> {
        if s.is_empty() {
            None
        } else {
            Some(NonEmptyString(s))
        }
    }

    fn as_str(&self) -> &str {
        &self.0
    }
}
```

### Zero Cost Verified

```rust
use std::mem::size_of;
assert_eq!(size_of::<u64>(), size_of::<UserId>()); // Both 8 bytes
```

The compiler generates identical machine code for operations on `UserId` and `u64`.

## Common Pitfalls

- Forgetting `#[repr(transparent)]` when the newtype is used in FFI.
- Implementing `Deref` to the inner type — this makes the newtype transparent to all methods, defeating the purpose.
- Not deriving `Clone`, `Copy`, `Hash` etc. when the inner type supports them.

## Best Practices

- Use newtypes for any domain ID, measurement, or validated string.
- Derive common traits rather than implementing them manually.
- Use `repr(transparent)` when FFI compatibility is needed.
- Avoid `Deref` to the inner type; provide explicit accessor methods instead.

## Summary

Newtypes provide compile-time type safety with zero runtime overhead. The compiler erases the wrapper entirely, generating the same machine code as the inner type. Use them liberally for domain modeling.

## Code Examples

**Newtype pattern with repr(transparent) for zero-cost type safety**

```rust
use std::mem::size_of;

#[repr(transparent)]
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
struct UserId(u64);

#[repr(transparent)]
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
struct OrderId(u64);

fn process_user(id: UserId) {
    println!("Processing user {}", id.0);
}

fn main() {
    let user = UserId(42);
    let order = OrderId(42);

    process_user(user);   // OK
    // process_user(order); // Compile error!

    // Zero cost: same size as u64
    assert_eq!(size_of::<UserId>(), size_of::<u64>());
    assert_eq!(size_of::<OrderId>(), size_of::<u64>());
}
```


## Resources

- [The Newtype Pattern in Rust](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#using-the-newtype-pattern-for-type-safety-and-abstraction) — The Rust Book's section on newtypes
- [repr(transparent) Reference](https://doc.rust-lang.org/reference/type-layout.html#the-transparent-representation) — Official documentation on transparent representation

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*