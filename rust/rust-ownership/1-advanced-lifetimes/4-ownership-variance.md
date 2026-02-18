---
source_course: "rust-ownership"
source_lesson: "rust-ownership-variance"
---

# Variance: Advanced Lifetime Rules

Variance describes how subtyping works with generics.

## Lifetime Subtyping

If `'long: 'short` (`'long` outlives `'short`), then `&'long T` can be used where `&'short T` is expected.

```rust
fn use_ref<'short>(r: &'short str) {
    println!("{r}");
}

let long_lived: &'static str = "hello";
use_ref(long_lived);  // OK! 'static: 'short
```

## Three Kinds of Variance

**Covariant:** Can substitute longer lifetime
- `&'a T` is covariant over `'a`
- `Box<T>` is covariant over `T`

**Contravariant:** Can substitute shorter lifetime (rare)
- `fn(T)` is contravariant over `T`

**Invariant:** Must be exactly the type
- `&'a mut T` is invariant over `T`
- `Cell<T>` is invariant over `T`

## Why &mut T is Invariant

If mutable references were covariant, this would be possible:

```rust
fn evil(s: &mut &'static str, short: &str) {
    *s = short;  // Would overwrite 'static with short-lived!
}

let mut s: &'static str = "hello";
let local = String::from("local");
// evil(&mut s, &local);  // If this worked...
// drop(local);
// println!("{s}");  // ...this would be use-after-free!
```

Invariance prevents this class of bugs.

See [Subtyping and Variance](https://doc.rust-lang.org/nomicon/subtyping.html).

## Code Examples

**Covariance vs Invariance**

```rust
// Covariance: &T can shrink lifetimes
fn covariant<'a>(long: &'a str) -> &'a str {
    let short: &str = long;  // OK: shrinking lifetime
    short
}

// Invariance: &mut T can't change lifetimes
fn invariant<'a>(r: &mut &'a str) {
    // Can't assign a different lifetime here
    let s: &'static str = "static";
    // *r = s;  // Error if 'a != 'static
}
```


## Resources

- [Subtyping and Variance](https://doc.rust-lang.org/nomicon/subtyping.html) — The Rustonomicon chapter on variance

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*