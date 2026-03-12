---
source_course: "rust-ownership"
source_lesson: "rust-ownership-lifetime-syntax"
---

# Lifetime Annotations & Elision

## Introduction
When the compiler cannot infer lifetime relationships on its own, you supply explicit lifetime annotations. Rust also has three elision rules that automatically insert lifetimes in common patterns, so most function signatures remain clean and annotation-free.

## Key Concepts
- **Lifetime parameter**: A generic parameter starting with `'` (e.g., `'a`, `'b`) that names a particular lifetime scope.
- **Lifetime elision**: A set of compiler rules that automatically assign lifetime parameters to function signatures in predictable patterns.
- **Struct lifetimes**: When a struct holds a reference, it must declare the lifetime of that reference so the borrow checker can track it.

## Real World Context
In production Rust code, you rarely see explicit lifetimes on simple functions because elision handles them. But the moment you write a function that returns a reference derived from multiple inputs, or a struct that borrows data, you need to understand the annotation syntax.

## Deep Dive
Lifetime parameters are declared in angle brackets and applied to references:

```rust
// Both inputs and output share the same lifetime
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

This says: "The returned reference will live at least as long as the shorter of x and y." When you need distinct lifetimes because the output only depends on one input, use separate parameters:

```rust
fn first_word<'a, 'b>(s: &'a str, _other: &'b str) -> &'a str {
    s.split_whitespace().next().unwrap_or(s)
}
```

The compiler applies three elision rules in order:

1. **Each input reference gets its own lifetime**: `fn foo(x: &i32, y: &i32)` becomes `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`.
2. **If exactly one input lifetime, it is assigned to all outputs**: `fn foo(x: &i32) -> &i32` becomes `fn foo<'a>(x: &'a i32) -> &'a i32`.
3. **If there is `&self` or `&mut self`, its lifetime is assigned to outputs**: `fn method(&self) -> &str` becomes `fn method<'a>(&'a self) -> &'a str`.

Structs that hold references must declare the lifetime:

```rust
struct Excerpt<'a> {
    part: &'a str,
}

impl<'a> Excerpt<'a> {
    // Elision rule 3: output gets lifetime of &self
    fn get(&self) -> &str {
        self.part
    }
}
```

### RPIT Lifetime Capture (2024 Edition)\n\n**RPIT** stands for Return Position Impl Trait — the `impl Trait` syntax in function return types.

In the 2024 edition, `impl Trait` in return position now captures **all** in-scope generic and lifetime parameters by default. In previous editions, lifetime parameters were not captured unless explicitly mentioned:

```rust
// 2021 edition: 'a is NOT captured — compiles
// 2024 edition: 'a IS captured — may cause borrow errors
fn make_iter<'a>(slice: &'a [i32]) -> impl Iterator<Item = &i32> {
    slice.iter()
}
```

If you need to opt out of capturing specific lifetimes in the 2024 edition, use the `use<>` syntax (stabilized in Rust 1.82):

```rust
// Explicitly capture only 'a, not other in-scope lifetimes
fn make_iter<'a, 'b>(a: &'a [i32], _b: &'b str) -> impl Iterator<Item = &'a i32> + use<'a> {
    a.iter()
}
```

This change makes lifetime capture more consistent with how generic type parameters were already captured.

## Common Pitfalls
1. **Adding unnecessary lifetime annotations** — If elision handles the signature, adding explicit lifetimes is noise. Only annotate when the compiler requires it.
2. **Confusing lifetime parameters with concrete lifetimes** — `'a` is not a fixed scope; it is a placeholder that the compiler fills in at each call site based on actual borrow scopes.
3. **Unexpected captures in the 2024 edition** — When migrating from the 2021 edition, `impl Trait` return types may capture more lifetimes than before, causing new borrow checker errors. Use `+ use<>` to restrict captures.

## Best Practices
1. **Learn the three elision rules** — Knowing when the compiler can infer lifetimes helps you write cleaner signatures and understand error messages.
2. **Use descriptive lifetime names in complex signatures** — For functions with many lifetime parameters, names like `'src` and `'dest` are clearer than `'a` and `'b`.
3. **Use `+ use<>` for precise control** — In the 2024 edition, when an `impl Trait` return type should not capture all in-scope lifetimes, use the explicit `use<>` syntax.

## Summary
- Lifetime annotations use the `'a` syntax and describe how reference lifetimes relate.
- Three elision rules automatically insert lifetimes in most function signatures.
- Structs holding references must declare the lifetime parameter.
- In the 2024 edition, `impl Trait` return types capture all in-scope lifetimes by default — use `+ use<>` to opt out.
- Only add explicit annotations when the compiler cannot infer them.

## Code Examples

**Struct and impl with lifetimes — elision rule 3 means the output lifetime of get() is inferred from &self**

```rust
// Struct with lifetime
struct Excerpt<'a> {
    part: &'a str,
}

impl<'a> Excerpt<'a> {
    // Elision rule 3: output gets lifetime of &self
    fn get(&self) -> &str {
        self.part
    }
    
    // Explicit when returning something else
    fn with_prefix(&self, prefix: &str) -> String {
        format!("{}{}", prefix, self.part)
    }
}
```


## Resources

- [Lifetime Elision](https://doc.rust-lang.org/reference/lifetime-elision.html) — Official Rust reference on lifetime elision rules

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*