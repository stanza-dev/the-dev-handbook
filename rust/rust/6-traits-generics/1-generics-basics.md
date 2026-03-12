---
source_course: "rust"
source_lesson: "rust-generics-basics"
---

# Generic Data Types

## Introduction

Without generics, you would need to write separate functions for every type: one for `i32`, one for `f64`, one for `String`, and so on. Generics let you write a single piece of code that works across many types, and Rust compiles it into type-specific code with zero runtime cost.

## Key Concepts

- **Type parameter**: A placeholder (conventionally `T`, `U`, etc.) that represents any type. It appears in angle brackets after the function, struct, or enum name.
- **Trait bound**: A constraint on a type parameter that says "T must implement this trait." Written as `T: TraitName` or with a `where` clause.
- **Monomorphization**: The compiler's process of replacing generic code with concrete, type-specific versions at compile time. This is why generics have zero runtime cost.

## Real World Context

Generics are the backbone of Rust's standard library. `Vec<T>`, `Option<T>`, `Result<T, E>`, and `HashMap<K, V>` are all generic types. Every time you write `Vec<String>` or `Result<User, DbError>`, you are using generics. Understanding them is essential for reading library APIs and writing reusable code.

## Deep Dive

A generic function declares type parameters in angle brackets. To do something useful like comparison, you add a trait bound:

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest { largest = item; }
    }
    largest
}
```

Structs can also be generic. A single type parameter means both fields share the same type:

```rust
struct Point<T> { x: T, y: T }

let int_point = Point { x: 5, y: 10 };     // Point<i32>
let float_point = Point { x: 1.0, y: 4.0 }; // Point<f64>
```

You can implement methods on generic structs, and also implement methods only for specific concrete types:

```rust
impl<T> Point<T> {
    fn x(&self) -> &T { &self.x }
}

impl Point<f64> {
    fn distance_from_origin(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

The `distance_from_origin` method only exists on `Point<f64>`, not on any other instantiation.

## Common Pitfalls

1. **Forgetting trait bounds** — A bare `T` has no capabilities beyond move/borrow/drop. If you try to compare, print, or clone a `T` without the appropriate trait bound, the compiler will reject it.
2. **Over-constraining** — Adding unnecessary trait bounds (e.g., requiring `Clone + Debug + Display`) when the function only needs `Display` limits reusability. Only require what you use.

## Best Practices

1. **Use `where` clauses for readability** — When you have multiple type parameters with multiple bounds, a `where` clause after the return type is cleaner than inline bounds.
2. **Start concrete, then generalize** — Write the function for a specific type first, then replace the concrete type with a type parameter and add the necessary bounds. This avoids over-engineering.

## Summary

- Generics let you write code that works across multiple types without duplication.
- Type parameters need trait bounds to perform any operation beyond moving or borrowing.
- Monomorphization means generics have zero runtime overhead.
- Use `where` clauses for complex bounds and start with concrete types before generalizing.

## Code Examples

**Multiple trait bounds**

```rust
// Generic function with multiple bounds
fn print_debug<T: std::fmt::Debug + Clone>(item: T) {
    let cloned = item.clone();
    println!("{:?}", cloned);
}

// Using where clause for readability
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: std::fmt::Display + Clone,
    U: Clone + std::fmt::Debug,
{
    // ...
    0
}
```


## Resources

- [Generic Data Types](https://doc.rust-lang.org/stable/book/ch10-01-syntax.html) — Official Rust Book chapter on generics in functions, structs, enums, and methods
- [Rust By Example: Generics](https://doc.rust-lang.org/rust-by-example/generics.html) — Hands-on examples of generic functions, implementations, trait bounds, and where clauses

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*