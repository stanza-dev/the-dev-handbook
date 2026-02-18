---
source_course: "rust-ownership"
source_lesson: "rust-ownership-unpin-trait"
---

# Unpin: Can Be Moved Even When Pinned

Most types implement `Unpin` automatically:

```rust
use std::marker::Unpin;

fn is_unpin<T: Unpin>() {}

is_unpin::<i32>();        // OK
is_unpin::<String>();     // OK
is_unpin::<Vec<i32>>();   // OK
```

For `Unpin` types, `Pin<&mut T>` is equivalent to `&mut T`:

```rust
use std::pin::Pin;

let mut x = 5;
let pinned: Pin<&mut i32> = Pin::new(&mut x);

// Can get mutable reference because i32: Unpin
let r: &mut i32 = Pin::into_inner(pinned);
*r = 10;
```

## !Unpin Types

Types that **cannot** be moved once pinned:

- Async function return types
- Types containing `PhantomPinned`
- Self-referential types

```rust
use std::marker::PhantomPinned;

struct NotUnpin {
    data: i32,
    _pin: PhantomPinned,  // Makes struct !Unpin
}

// Can't move out of Pin<Box<NotUnpin>>
```

## Why Futures are !Unpin

Async blocks capture local variables that may reference each other:

```rust
async fn has_self_ref() {
    let data = [1, 2, 3];
    let slice = &data[..];  // Reference into local
    yield_now().await;       // Suspension point
    println!("{:?}", slice); // Still uses reference
}
// The generated Future contains both data AND slice
// Moving it would invalidate slice
```

## Code Examples

**Unpin vs !Unpin futures**

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

// Most types: Unpin
struct SimpleFuture;

impl Future for SimpleFuture {
    type Output = i32;
    fn poll(self: Pin<&mut Self>, _cx: &mut Context) -> Poll<i32> {
        Poll::Ready(42)
    }
}

// Self-referential: !Unpin
struct ComplexFuture {
    data: String,
    slice: *const str,
    _pin: std::marker::PhantomPinned,
}

// For !Unpin, poll() is the only way to access &mut self
// This prevents accidentally moving the future
```


---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*