---
source_course: "rust-ownership"
source_lesson: "rust-ownership-variance"
---

# Subtyping & Variance

## Introduction
Variance describes how the subtyping relationship between lifetimes interacts with generic type constructors like `&T`, `&mut T`, and `Cell<T>`. It is one of the most nuanced topics in Rust's type system and is essential for understanding why certain patterns with mutable references are rejected.

## Key Concepts
- **Subtyping**: If `'long: 'short` (meaning `'long` outlives `'short`), then `&'long T` is a subtype of `&'short T`.
- **Covariance**: A type constructor `F<T>` is covariant over `T` if `F<Sub>` is a subtype of `F<Super>` whenever `Sub` is a subtype of `Super`.
- **Contravariance**: `F<T>` is contravariant over `T` if the subtyping relationship reverses.
- **Invariance**: `F<T>` is invariant over `T` if no subtyping relationship exists regardless of T's subtypes.

## Real World Context
Variance is why you can pass a `&'static str` where a `&'a str` is expected (covariance of `&T`), but you cannot assign a short-lived reference through a `&mut &'long str` (invariance of `&mut T`). Understanding variance prevents confusion when the compiler rejects code that "looks correct" at first glance.

## Deep Dive
Lifetime subtyping is the foundation: if `'long` outlives `'short`, then a `&'long T` can be used wherever a `&'short T` is expected:

```rust
fn use_ref<'short>(r: &'short str) {
    println!("{r}");
}

let long_lived: &'static str = "hello";
use_ref(long_lived);  // OK! 'static: 'short
```

Shared references `&'a T` are covariant over both `'a` and `T`. This means you can "shrink" lifetimes freely when passing shared references.

Mutable references `&'a mut T` are covariant over `'a` but **invariant** over `T`. This invariance is critical for soundness. Consider what would happen if `&mut T` were covariant over `T`:

```rust
fn evil(r: &mut &'static str) {
    // If &mut T were covariant, we could call this with
    // &mut &'short str coerced to &mut &'static str
}

fn unsound<'short>(r: &mut &'short str, short: &'short str) {
    // If this were allowed through covariance:
    // *r = short;
    // Then after 'short ends, *r would be dangling!
}
```

The correct demonstration of invariance is that you cannot pass a `&mut &'short str` where `&mut &'long str` is expected:

```rust
fn needs_static_ref(r: &mut &'static str) {
    // This function expects a mutable reference to a 'static str
}

fn demo<'a>(r: &mut &'a str) {
    // Cannot call needs_static_ref(r) because
    // &mut &'a str is NOT a subtype of &mut &'static str
    // even though &'static str IS a subtype of &'a str.
    // That's invariance: &mut T does not permit T to vary.
}
```

Note that covariance of `&T` means assigning `&'static str` into `&'a str` is always fine — a longer-lived reference can substitute for a shorter-lived one. Invariance of `&mut T` prevents the reverse direction exploit.

## Common Pitfalls
1. **Thinking `&mut T` prevents all lifetime coercion** — `&'a mut T` is still covariant over `'a` (you can shorten the borrow duration). It is only invariant over `T` itself.
2. **Confusing covariance with the direction of assignment** — Covariance means a *subtype* can substitute for a *supertype*. For lifetimes, longer is the subtype.

## Best Practices
1. **Trust the compiler** — If the borrow checker rejects a lifetime relationship involving `&mut`, it is almost certainly preventing a soundness hole. Restructure the code rather than fighting it.
2. **Use the Rustonomicon as a reference** — The Rustonomicon's chapter on subtyping and variance is the authoritative resource for these rules.

## Summary
- `&'a T` is covariant: longer lifetimes can substitute for shorter ones.
- `&'a mut T` is invariant over `T`: you cannot change the inner type's lifetime through a mutable reference.
- Invariance prevents a class of use-after-free bugs involving mutable references.
- `fn(T)` is contravariant over `T`, though this is rare in practice.

## Code Examples

**Covariance vs Invariance — shared references allow lifetime shrinking, mutable references do not allow the inner type to vary**

```rust
// Covariance: &'static str can be used as &'a str
fn takes_short<'a>(r: &'a str) -> &'a str {
    r
}

let long: &'static str = "hello";
let result = takes_short(long); // OK: 'static outlives 'a

// Invariance: &mut &'a str cannot vary the inner lifetime
fn needs_static(r: &mut &'static str) {
    *r = "still static";
}

// This would NOT compile:
// fn try_coerce<'a>(r: &mut &'a str) {
//     needs_static(r); // Error! &mut &'a str != &mut &'static str
// }
```


## Resources

- [Subtyping and Variance](https://doc.rust-lang.org/nomicon/subtyping.html) — The Rustonomicon chapter on variance

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*