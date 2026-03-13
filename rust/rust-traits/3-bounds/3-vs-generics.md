---
source_course: "rust-traits"
source_lesson: "rust-traits-impl-trait-vs-generics"
---

# impl Trait vs Generic Parameters

## Introduction
Rust offers two syntaxes for trait bounds in function arguments: `impl Trait` (argument-position impl Trait, or APIT) and explicit generic parameters. While they are equivalent in many cases, there are important differences that affect API design, especially around turbofish syntax and type inference.

## Key Concepts
- **APIT (Argument-Position impl Trait)**: Syntactic sugar that creates an anonymous generic parameter: `fn foo(x: impl Clone)`.
- **Explicit Generic**: A named generic parameter: `fn foo<T: Clone>(x: T)`.
- **Turbofish**: The `::<Type>` syntax for specifying generic parameters explicitly. Not available with APIT.

## Real World Context
Public library APIs must choose between APIT and explicit generics carefully. APIT is simpler for simple cases, but explicit generics are required when you need to name the type, use it multiple times, or let callers specify it with turbofish.

## Deep Dive

### Equivalence

For simple single-use parameters, they are equivalent:

```rust
fn print_it(item: impl Display) { println!("{item}"); }
fn print_it<T: Display>(item: T) { println!("{item}"); }
```

### When Explicit Generics Are Required

**Multiple parameters of the same type:**

```rust
// These are NOT equivalent:
fn same_type<T: Clone>(a: T, b: T) { } // a and b must be the same type
fn diff_type(a: impl Clone, b: impl Clone) { } // a and b can be different types

same_type(1_i32, 2_i32); // OK
// same_type(1_i32, "hi"); // Error: mismatched types
diff_type(1_i32, "hi");   // OK: different anonymous types
```

**Turbofish syntax:**

```rust
fn create<T: Default>() -> T { T::default() }
let x = create::<String>(); // Turbofish works with named generics

// fn create_impl() -> impl Default { ... } // Can't turbofish this
```

**Returning the same type as input:**

```rust
fn identity<T>(x: T) -> T { x } // Return type matches input
// fn identity(x: impl Clone) -> ??? { x } // Can't name the return type
```

### When to Use APIT

Prefer APIT for simple, single-use bounds in private or internal functions:

```rust
fn log(msg: impl Display) { println!("[LOG] {msg}"); }
fn process(iter: impl Iterator<Item = i32>) -> i32 { iter.sum() }
```

## Common Pitfalls
1. **Using APIT when the type must be named** — If you need the same type in multiple positions (arguments, return type, struct fields), you need explicit generics.
2. **Assuming APIT enables dynamic dispatch** — `impl Trait` in argument position is still static dispatch. For dynamic dispatch, use `&dyn Trait`.

## Best Practices
1. **Use APIT for simple leaf functions** — When the type appears once and is not returned, APIT is cleaner.
2. **Use explicit generics for public APIs** — Named generics let callers use turbofish and make documentation clearer.

## Summary
- `impl Trait` in arguments creates an anonymous generic parameter.
- Explicit generics are required for turbofish, repeated types, and return types.
- `fn foo(a: impl T, b: impl T)` allows different types; `fn foo<T>(a: T, b: T)` requires the same type.
- Prefer APIT for simple cases, explicit generics for public APIs.

## Code Examples

**APIT works for simple single-use bounds, but explicit generics are needed when the type must be named, repeated, or specified with turbofish**

```rust
use std::fmt::Display;

// APIT: simple, anonymous generic
fn log(msg: impl Display) {
    println!("[LOG] {msg}");
}

// Explicit generic: needed when type appears multiple times
fn max_of<T: PartialOrd>(a: T, b: T) -> T {
    if a > b { a } else { b }
}

// Both called the same way
log("server started");
log(42);
let m = max_of(10, 20); // T inferred as i32
let m = max_of::<f64>(1.5, 2.5); // Turbofish works with named generics
```


## Resources

- [impl Trait](https://doc.rust-lang.org/reference/types/impl-trait.html) — Rust Reference on impl Trait in argument and return position

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*