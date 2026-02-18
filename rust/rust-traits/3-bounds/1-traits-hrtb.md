---
source_course: "rust-traits"
source_lesson: "rust-traits-hrtb"
---

# HRTB: for<'a>

Higher-Ranked Trait Bounds express "for any lifetime."

## The Problem

```rust
fn call_with_ref<F>(f: F)
where
    F: Fn(&i32),  // What lifetime?
{
    let x = 42;
    f(&x);  // Lifetime of &x is local to this function
}
```

## The Solution: for<'a>

```rust
fn call_with_ref<F>(f: F)
where
    F: for<'a> Fn(&'a i32),  // Works for ANY lifetime 'a
{
    let x = 42;
    f(&x);  // ✓ Works!
}
```

## When You'll See HRTB

Most of the time, Rust infers HRTB for you:

```rust
// These are equivalent:
fn foo<F: Fn(&i32)>(f: F) { }           // Sugar
fn foo<F: for<'a> Fn(&'a i32)>(f: F) { }  // Explicit
```

But sometimes you need to be explicit:

```rust
// Storing a closure that takes references
struct CallbackHolder<F> 
where
    F: for<'a> Fn(&'a str) -> &'a str,
{
    callback: F,
}
```

## HRTB in Practice

```rust
use std::fmt::Debug;

// Function that works with any printable reference
fn process<F>(f: F) 
where
    F: for<'a> Fn(&'a dyn Debug),
{
    f(&42);
    f(&"hello");
    f(&vec![1, 2, 3]);
}

process(|x| println!("{x:?}"));
```

See [HRTB](https://doc.rust-lang.org/nomicon/hrtb.html).

## Code Examples

**HRTB in parser combinators**

```rust
// Real example: Parser combinators
trait Parser {
    type Output;
    fn parse<'a>(&self, input: &'a str) -> Option<(Self::Output, &'a str)>;
}

// Combinator that transforms parser output
fn map<P, F, B>(parser: P, f: F) -> Map<P, F>
where
    P: Parser,
    F: for<'a> Fn(P::Output) -> B,  // Works for any parse result lifetime
{
    Map { parser, f }
}

struct Map<P, F> {
    parser: P,
    f: F,
}
```


## Resources

- [Higher-Ranked Trait Bounds](https://doc.rust-lang.org/nomicon/hrtb.html) — Rustonomicon on HRTB

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*