---
source_course: "rust-ownership"
source_lesson: "rust-ownership-pin-practical"
---

# Working with Pin

## Creating Pinned Values

```rust
use std::pin::Pin;

// On heap with Box::pin
let pinned: Pin<Box<MyType>> = Box::pin(MyType::new());

// On stack with pin! macro (Rust 1.68+)
use std::pin::pin;

let mut future = pin!(async { 42 });
let result = future.as_mut().poll(cx);
```

## Pin Projection

Accessing fields of a pinned struct requires care:

```rust
struct Wrapper {
    inner: InnerType,
    other: OtherType,
}

impl Wrapper {
    // Project Pin to inner field
    fn inner(self: Pin<&mut Self>) -> Pin<&mut InnerType> {
        // Only safe if InnerType is structurally pinned
        unsafe { self.map_unchecked_mut(|s| &mut s.inner) }
    }
    
    // Non-pinned field can be accessed normally
    fn other(self: Pin<&mut Self>) -> &mut OtherType {
        unsafe { &mut self.get_unchecked_mut().other }
    }
}
```

## The pin-project Crate

Makes pin projection safe and easy:

```rust
use pin_project::pin_project;

#[pin_project]
struct MyFuture {
    #[pin]  // This field is pinned
    inner: SomeFuture,
    count: usize,  // This field is not pinned
}

impl Future for MyFuture {
    type Output = ();
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context) -> Poll<()> {
        let this = self.project();
        // this.inner: Pin<&mut SomeFuture>
        // this.count: &mut usize
        this.inner.poll(cx)
    }
}
```

See [pin-project](https://docs.rs/pin-project/latest/pin_project/).

## Code Examples

**Stack-pinned future polling**

```rust
use std::future::Future;
use std::pin::pin;
use std::task::{Context, Poll};

// Polling a future on the stack
async fn do_work() -> i32 {
    // some async work
    42
}

fn block_on<F: Future>(future: F) -> F::Output {
    let mut future = pin!(future);
    let waker = /* create waker */;
    let mut cx = Context::from_waker(&waker);
    
    loop {
        match future.as_mut().poll(&mut cx) {
            Poll::Ready(output) => return output,
            Poll::Pending => { /* wait for waker */ }
        }
    }
}
```


## Resources

- [pin-project crate](https://docs.rs/pin-project/latest/pin_project/) — Safe pin projections

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*