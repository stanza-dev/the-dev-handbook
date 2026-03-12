---
source_course: "rust"
source_lesson: "rust-traits-intro"
---

# Traits: Defining Shared Behavior

## Introduction

Traits are Rust's mechanism for defining shared behavior across types. If you are familiar with interfaces in Java or TypeScript, traits serve a similar role — but with more power, including default implementations and blanket impls.

## Key Concepts

- **Trait**: A collection of method signatures (and optionally default implementations) that types can implement. Defined with the `trait` keyword.
- **impl Trait for Type**: The syntax to implement a trait for a specific type. Each required method must be provided.
- **Default implementation**: A method body provided in the trait definition itself. Types can override it or use it as-is.
- **Trait bounds**: Constraints that say "this type must implement trait X." Used in generic functions and structs.

## Real World Context

Traits are the foundation of Rust's polymorphism. The standard library defines dozens of traits like `Display`, `Iterator`, `From`, and `Clone`. When you write `println!("{}", x)`, you are relying on `x` implementing `Display`. Traits also power operator overloading, serialization with serde, and async runtimes like tokio.

## Deep Dive

Define a trait with method signatures:

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

Then implement it for your types:

```rust
pub struct Article {
    pub title: String,
    pub author: String,
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}, by {}", self.title, self.author)
    }
}
```

Default implementations let you provide shared behavior that types can optionally override:

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

Traits as function parameters use `impl Trait` or explicit trait bounds:

```rust
fn notify(item: &impl Summary) {
    println!("Breaking: {}", item.summarize());
}
// Equivalent trait bound syntax:
fn notify_generic<T: Summary>(item: &T) { /* ... */ }
```

You can also return a type that implements a trait using `-> impl Trait`, which hides the concrete type behind a trait interface.

## Common Pitfalls

1. **Orphan rule violations** — You can only implement a trait for a type if either the trait or the type is defined in your crate. You cannot implement a foreign trait for a foreign type. The workaround is the newtype pattern.
2. **Confusing `impl Trait` return with `dyn Trait`** — `-> impl Trait` returns a single concrete type known at compile time. `-> Box<dyn Trait>` returns a trait object allowing different concrete types at runtime. They are not interchangeable.

## Best Practices

1. **Provide defaults when sensible** — Default implementations reduce boilerplate for implementors. Only require methods that truly differ between types.
2. **Prefer `impl Trait` over `dyn Trait`** — Static dispatch with `impl Trait` has zero overhead. Use `dyn Trait` only when you need runtime polymorphism (e.g., storing mixed types in a collection).

## Summary

- Traits define shared behavior with method signatures and optional defaults.
- `impl TraitName for Type` provides the concrete implementation.
- Use `impl Trait` in function parameters and return types for zero-cost polymorphism.
- The orphan rule governs where trait implementations can live.
- Default implementations reduce boilerplate while allowing customization.

## Code Examples

**Blanket implementations — the standard library auto-implements ToString for any type with Display**

```rust
use std::fmt::Display;

// This blanket impl exists in the standard library.
// You cannot write this yourself (orphan rules prevent it),
// but it means any type implementing Display gets ToString for free.
//
// impl<T: Display> ToString for T {
//     fn to_string(&self) -> String { ... }
// }

// Because i32 implements Display, it automatically has .to_string()
let s = 3.to_string();
println!("{s}"); // Output: 3
```


## Resources

- [Traits](https://doc.rust-lang.org/stable/book/ch10-02-traits.html) — Complete guide to traits

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*