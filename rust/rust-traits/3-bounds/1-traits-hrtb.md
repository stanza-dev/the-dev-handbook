---
source_course: "rust-traits"
source_lesson: "rust-traits-hrtb"
---

# Higher-Ranked Trait Bounds

## Introduction
Higher-Ranked Trait Bounds (HRTB) express "for any lifetime" using the `for<'a>` syntax. They are essential when you need a function or closure that works with references of any lifetime, not just one specific lifetime.

## Key Concepts
- **HRTB (`for<'a>`)**: A universally quantified lifetime bound meaning the trait must hold for every possible lifetime.
- **Lifetime Elision**: Rust often infers HRTB automatically for closure bounds, so you rarely need to write `for<'a>` explicitly.
- **Universal Quantification**: The `for<'a>` syntax means the bound is not tied to any particular lifetime.

## Real World Context
Parser combinator libraries, callback storage patterns, and any API that stores closures operating on borrowed data rely on HRTB. Without them, you cannot express "this closure works with any reference lifetime."

## Deep Dive
Consider a function that calls a closure with a local reference:

```rust
fn call_with_ref<F>(f: F)
where
    F: for<'a> Fn(&'a i32),  // Works for ANY lifetime
{
    let x = 42;
    f(&x); // Lifetime of &x is local — HRTB handles this
}
```

Most of the time, Rust infers HRTB when you write closure bounds:

```rust
// These are equivalent:
fn foo<F: Fn(&i32)>(f: F) { }            // Sugar — Rust infers for<'a>
fn foo<F: for<'a> Fn(&'a i32)>(f: F) { }  // Explicit HRTB
```

You need explicit HRTB when storing closures in structs:

```rust
struct CallbackHolder<F>
where
    F: for<'a> Fn(&'a str) -> &'a str,
{
    callback: F,
}
```

Without `for<'a>`, the struct would need a lifetime parameter, tying it to a specific borrow.

## Common Pitfalls
1. **Confusing `for<'a>` with a named lifetime** — `for<'a>` is not a specific lifetime; it means "for all lifetimes." It is universal, not existential.
2. **Adding explicit HRTB unnecessarily** — Rust's elision rules handle most cases. Only write `for<'a>` when the compiler tells you it is needed.

## Best Practices
1. **Let elision handle closure bounds** — Write `Fn(&str)` and let Rust infer the HRTB. Only spell out `for<'a>` when storing closures or when the compiler requires it.
2. **Use HRTB for callback storage** — When a struct holds a closure that operates on borrows, explicit HRTB is the correct tool.

## Summary
- `for<'a>` means the bound holds for every possible lifetime.
- Rust infers HRTB for most closure parameters automatically.
- Explicit HRTB is needed when storing closures in structs or when lifetime elision is insufficient.
- HRTB is universal quantification over lifetimes.

## Code Examples

**HRTB lets a function accept closures that work with references of any lifetime, enabling the closure to be called with different borrow scopes**

```rust
use std::fmt::Debug;

// HRTB: function accepts any closure that works with any reference lifetime
fn apply_to_values<F>(f: F)
where
    F: for<'a> Fn(&'a dyn Debug),
{
    f(&42);
    f(&"hello");
    f(&vec![1, 2, 3]);
}

fn main() {
    apply_to_values(|x| println!("{x:?}"));
    // The closure works with &i32, &&str, and &Vec<i32>
    // because the HRTB ensures it handles any lifetime
}
```


## Resources

- [Higher-Ranked Trait Bounds](https://doc.rust-lang.org/nomicon/hrtb.html) — Rustonomicon chapter explaining HRTB and universal lifetime quantification

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*