---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-send-trait"
---

# Send: Safe to Transfer Ownership

A type is `Send` if it's safe to move to another thread.

```rust
use std::thread;

fn requires_send<T: Send>(t: T) {
    thread::spawn(move || {
        drop(t);
    });
}
```

## What's NOT Send?

- **`Rc<T>`**: Non-atomic reference count
- **`*const T`, `*mut T`**: Raw pointers
- **Types with thread-local state**

## Auto-Implementation

`Send` is automatically implemented if all fields are `Send`:

```rust
struct MySendType {
    a: String,      // Send ✓
    b: Arc<i32>,    // Send ✓
} // MySendType is Send ✓

struct NotSend {
    rc: Rc<i32>,    // NOT Send ✗
} // NotSend is NOT Send ✗
```

## Why Rc is !Send

```rust
use std::rc::Rc;

let rc = Rc::new(5);
let rc2 = Rc::clone(&rc);

// If we could send rc to another thread...
// thread::spawn(move || {
//     drop(rc);  // Decrements count non-atomically
// });
// drop(rc2);  // Race condition! Both decrement at same time
```

## Unsafe Manual Implementation

```rust
struct MyUnsafeType(*mut u8);

// DANGER: Only do this if you're sure it's safe!
unsafe impl Send for MyUnsafeType {}
```

See [Send and Sync](https://doc.rust-lang.org/nomicon/send-and-sync.html).

## Code Examples

**spawn requires Send**

```rust
use std::thread;

// Thread::spawn bounds
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
where
    F: FnOnce() -> T + Send + 'static,
    T: Send + 'static,
{
    // Both the closure AND return value must be Send
}

// This works:
let s = String::from("hello");
thread::spawn(move || {
    s.len()  // String: Send, usize: Send
});

// This doesn't:
// let rc = Rc::new(5);
// thread::spawn(move || *rc);  // Rc: !Send
```


---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*