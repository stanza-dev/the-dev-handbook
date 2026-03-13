---
source_course: "rust-traits"
source_lesson: "rust-traits-impl-trait"
---

# impl Trait: Opaque Types

## Introduction
`impl Trait` lets you hide the concrete return type of a function while keeping static dispatch. Since Rust 1.87, you can also use precise capturing (`use<...>`) in trait definitions to control which generic parameters an opaque type captures.

## Key Concepts
- **Opaque Type**: A type whose concrete identity is hidden behind `impl Trait`, known only to the compiler.
- **Return-Position `impl Trait` (RPIT)**: Used in function return types to hide complex types.
- **Argument-Position `impl Trait` (APIT)**: Syntactic sugar for generic type parameters in function arguments.
- **Precise Capturing**: The `use<...>` syntax (Rust 1.87) controls which lifetimes and type parameters an opaque type captures.

## Real World Context
Without `impl Trait`, returning an iterator chain would require spelling out the full type: `Filter<Map<Range<i32>, fn(i32) -> i32>, fn(&i32) -> bool>`. RPIT makes this ergonomic. Closures are unspeakable types that can only be returned via `impl Fn`.

## Deep Dive

### Return Position

```rust
fn make_iter() -> impl Iterator<Item = i32> {
    (0..10).map(|x| x * 2).filter(|x| *x > 5)
}
// The caller sees "some Iterator<Item = i32>" but cannot name the type
```

This is essential for returning closures, since each closure has a unique anonymous type:

```rust
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n
}
let add_five = make_adder(5);
println!("{}", add_five(10)); // 15
```

### Argument Position

In argument position, `impl Trait` is sugar for a generic parameter:

```rust
// These are equivalent:
fn print_items(iter: impl Iterator<Item = i32>) { /* ... */ }
fn print_items<I: Iterator<Item = i32>>(iter: I) { /* ... */ }
```

### impl Trait vs dyn Trait

| Feature | `impl Trait` | `dyn Trait` |
|---------|-------------|-------------|
| Dispatch | Static | Dynamic |
| Size known | Compile time | Runtime (fat pointer) |
| Heterogeneous | No | Yes |
| Performance | Fastest (inlinable) | Vtable overhead |

### Limitations

RPIT requires all return paths to produce the same concrete type:

```rust
// Error: different concrete types
fn bad(cond: bool) -> impl Display {
    if cond { 42 } else { "hi" } // i32 vs &str!
}

// Use dyn Trait for heterogeneous returns:
fn good(cond: bool) -> Box<dyn Display> {
    if cond { Box::new(42) } else { Box::new("hi") }
}
```

Note: Type Alias Impl Trait (`type Foo = impl Bar;`) is **not yet stable** and remains a nightly-only feature.

## Common Pitfalls
1. **Trying to return different types from branches** — `impl Trait` is a single concrete type. Use `Box<dyn Trait>` when you need to return different types conditionally.
2. **Assuming TAIT is stable** — `type Alias = impl Trait;` in type aliases is not yet stabilized. Use concrete types or `Box<dyn Trait>` for type aliases.

## Best Practices
1. **Prefer RPIT for iterator and closure return types** — It avoids spelling out complex nested types and keeps signatures clean.
2. **Use precise capturing (`use<...>`) in trait definitions** — Since Rust 1.87, `-> impl Trait + use<'a, T>` lets you control exactly which generic parameters are captured by the opaque type.

## Summary
- `impl Trait` provides opaque types with static dispatch.
- In return position, it hides complex concrete types like iterator chains and closures.
- In argument position, it is sugar for a generic type parameter.
- Type Alias Impl Trait (TAIT) remains nightly-only as of Rust 1.94.
- Precise capturing (`use<...>`) was stabilized in Rust 1.87 for trait definitions.

## Code Examples

**impl Trait hides unspeakable types like closures and iterator chains, letting you return them from functions without naming the full concrete type**

```rust
// Returning closures with impl Trait
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n
}

let add_five = make_adder(5);
println!("{}", add_five(10));  // 15

// Returning complex iterator chains
fn evens_doubled(max: i32) -> impl Iterator<Item = i32> {
    (0..max).filter(|n| n % 2 == 0).map(|n| n * 2)
}

for val in evens_doubled(10) {
    println!("{val}"); // 0, 4, 8, 12, 16
}
```


## Resources

- [impl Trait](https://doc.rust-lang.org/reference/types/impl-trait.html) — Official Rust Reference on impl Trait in argument and return position

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*