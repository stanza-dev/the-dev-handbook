---
source_course: "rust-ownership"
source_lesson: "rust-ownership-unpin-trait"
---

# The Unpin Trait

## Introduction
Unpin is an auto trait that indicates a type is safe to move even after being pinned. Most standard types implement Unpin, which means Pin has no effect on them. Understanding Unpin is key to knowing when Pin actually matters and when it is a no-op.

## Key Concepts
- **Unpin**: An auto trait implemented by most types, indicating the type has no self-references and can be safely moved out of Pin.
- **!Unpin**: Types that do NOT implement Unpin — they genuinely cannot be moved once pinned. This includes async block return types and types containing PhantomPinned.
- **Pin<&mut T> where T: Unpin**: Equivalent to `&mut T` — Pin has no effect because the type opts into being movable.

## Real World Context
When writing manual Future implementations or building async combinators, you encounter Pin<&mut Self> in the poll method. For most custom futures that do not hold self-references, your type is Unpin and you can freely access `&mut self`. Only when you store other !Unpin futures (like async block results) do you need to handle Pin projection carefully.

## Deep Dive
Most types implement Unpin automatically:

```rust
use std::marker::Unpin;

fn is_unpin<T: Unpin>() {}

is_unpin::<i32>();        // OK
is_unpin::<String>();     // OK
is_unpin::<Vec<i32>>();   // OK
```

For Unpin types, Pin<&mut T> is equivalent to &mut T. You can freely extract the inner reference:

```rust
use std::pin::Pin;

let mut x = 5;
let pinned: Pin<&mut i32> = Pin::new(&mut x);

// Can get mutable reference because i32: Unpin
let r: &mut i32 = Pin::into_inner(pinned);
*r = 10;
```

Types that are !Unpin genuinely cannot be moved:

- Async function/block return types
- Types containing `PhantomPinned`
- Self-referential types

```rust
use std::marker::PhantomPinned;

struct NotUnpin {
    data: i32,
    _pin: PhantomPinned,  // Makes struct !Unpin
}

// Can't get &mut NotUnpin from Pin<&mut NotUnpin>
// Can't move out of Pin<Box<NotUnpin>>
```

Async blocks capture local variables that may reference each other across await points:

```rust
async fn has_self_ref() {
    let data = [1, 2, 3];
    let slice = &data[..];       // Reference into local data
    tokio::task::yield_now().await;  // Suspension point
    println!("{:?}", slice);     // Still uses the reference
}
// The generated Future contains both data AND slice
// Moving it would invalidate slice — hence it's !Unpin
```

## Common Pitfalls
1. **Assuming all futures are !Unpin** — Simple futures that don't hold cross-await references may be Unpin. Only futures with self-references are !Unpin.
2. **Fighting Pin when your type is Unpin** — If your custom type implements Unpin (the default), you can use `Pin::into_inner()` to get a regular `&mut` reference.

## Best Practices
1. **Check if your type is Unpin before adding Pin complexity** — Use `fn assert_unpin<T: Unpin>() {}` to verify at compile time.
2. **Use Box::pin to erase !Unpin concerns** — `Box::pin(future)` returns `Pin<Box<F>>`, which is always valid regardless of whether F is Unpin.

## Summary
- Most types are Unpin, meaning Pin has no effect on them.
- Async block return types are !Unpin because they may contain self-references.
- Pin<&mut T> where T: Unpin is equivalent to &mut T.
- PhantomPinned makes a type !Unpin, opting into genuine move prevention.

## Code Examples

**Unpin vs !Unpin futures — SimpleFuture can be moved freely, ComplexFuture cannot**

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

// Most types: Unpin — Pin has no special effect
struct SimpleFuture;

impl Future for SimpleFuture {
    type Output = i32;
    fn poll(self: Pin<&mut Self>, _cx: &mut Context) -> Poll<i32> {
        Poll::Ready(42)
    }
}

// Self-referential: !Unpin — Pin prevents moving
struct ComplexFuture {
    data: String,
    slice: *const str,
    _pin: std::marker::PhantomPinned,
}

// For !Unpin, poll() is the only way to access &mut self
// This prevents accidentally moving the future
```


## Resources

- [Unpin Documentation](https://doc.rust-lang.org/std/marker/trait.Unpin.html) — Official API reference for the Unpin marker trait

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*