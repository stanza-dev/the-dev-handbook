---
source_course: "rust-traits"
source_lesson: "rust-traits-supertraits"
---

# Supertraits & Blanket Impls

## Introduction
Supertraits let you build trait hierarchies: requiring that a type implements one trait before it can implement another. Blanket implementations extend traits to all types matching a bound. Together, they form the backbone of Rust's composable trait system.

## Key Concepts
- **Supertrait**: A trait required by another trait, declared with `trait A: B { }`. Implementing `A` requires first implementing `B`.
- **Blanket Implementation**: An `impl<T: Bound> Trait for T` that applies to all types matching the bound.
- **Trait Coherence**: The orphan rule ensures that blanket impls do not conflict with local impls.

## Real World Context
`Error: Debug + Display` ensures every error type is printable. The blanket impl `impl<T: Display> ToString for T` gives every displayable type a `.to_string()` method for free.

## Deep Dive

### Supertraits

Declare that implementing one trait requires another:

```rust
use std::fmt::{Debug, Display};

trait Error: Debug + Display {
    fn source(&self) -> Option<&(dyn Error + 'static)> { None }
}

// To implement Error, you MUST implement Debug + Display
#[derive(Debug)]
struct MyError(String);

impl Display for MyError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{}", self.0)
    }
}

impl Error for MyError {} // OK: Debug + Display are both implemented
```

### Blanket Implementations

Implement a trait for all types matching a bound:

```rust
impl<T: Display> ToString for T {
    fn to_string(&self) -> String { format!("{self}") }
}
// Now 42.to_string(), 3.14.to_string() etc. all work
```

The `From`/`Into` blanket impl is another classic:

```rust
impl<T, U> Into<U> for T where U: From<T> {
    fn into(self) -> U { U::from(self) }
}
// Implement From<A> for B, and you get Into<B> for A for free
```

## Common Pitfalls
1. **Orphan rule conflicts with blanket impls** — You cannot implement a foreign trait for a foreign type. Blanket impls in upstream crates can prevent you from implementing traits on your types.
2. **Circular supertrait requirements** — Traits cannot form cycles. If `A: B` and `B: A`, the compiler rejects it.

## Best Practices
1. **Keep supertrait hierarchies shallow** — Deep hierarchies force implementors to satisfy many traits. Prefer composition over deep inheritance.
2. **Provide blanket impls for free functionality** — If your trait has a method that can be derived from a simpler trait, provide a blanket impl.

## Summary
- Supertraits enforce trait dependencies: `trait A: B` means implementing A requires B.
- Blanket impls provide automatic implementations for all types matching a bound.
- `Display` → `ToString` and `From` → `Into` are canonical blanket impl examples.
- Keep trait hierarchies shallow and composable.

## Code Examples

**A Loggable supertrait requiring Debug, with a blanket impl that gives every Debug type a .log() method for free**

```rust
use std::fmt::Debug;

// Custom supertrait + blanket implementation
trait Loggable: Debug {
    fn log(&self) {
        println!("[LOG] {:?}", self);
    }
}

// Blanket impl: everything that is Debug becomes Loggable
impl<T: Debug> Loggable for T {}

fn main() {
    42.log();                // [LOG] 42
    "hello".log();           // [LOG] "hello"
    vec![1, 2, 3].log();    // [LOG] [1, 2, 3]
    Some(true).log();        // [LOG] Some(true)
}
```


## Resources

- [Supertraits](https://doc.rust-lang.org/book/ch20-03-advanced-traits.html) — Rust Book section on supertraits and advanced trait patterns

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*