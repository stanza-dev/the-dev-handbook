---
source_course: "rust-ownership"
source_lesson: "rust-ownership-refcell-basics"
---

# RefCell<T>: Runtime Borrow Checking

## Introduction
RefCell<T> moves Rust's borrow checking from compile time to runtime, allowing you to mutate data through shared references for any type — not just Copy types. The tradeoff is that violating the borrow rules causes a runtime panic instead of a compile error.

## Key Concepts
- **RefCell<T>**: A container that enforces Rust's borrowing rules at runtime, panicking if they are violated.
- **Ref<T>**: A smart pointer returned by `borrow()` that acts like `&T` and tracks the immutable borrow.
- **RefMut<T>**: A smart pointer returned by `borrow_mut()` that acts like `&mut T` and tracks the mutable borrow.

## Real World Context
RefCell is commonly used in test mocks where a trait method takes `&self` but the mock needs to record calls internally. It is also essential for the `Rc<RefCell<T>>` pattern that enables shared mutable state in single-threaded graph structures.

## Deep Dive
RefCell provides `borrow()` and `borrow_mut()` methods that track borrows at runtime:

```rust
use std::cell::RefCell;

let x = RefCell::new(5);

{
    let mut borrow = x.borrow_mut();
    *borrow += 1;
}  // Mutable borrow ends when guard drops

println!("{}", x.borrow());  // 6
```

The runtime borrow rules mirror the compile-time rules:

| Method | Returns | Panics if |
|--------|---------|----------|
| `borrow()` | `Ref<T>` | Already mutably borrowed |
| `borrow_mut()` | `RefMut<T>` | Already borrowed (any) |
| `try_borrow()` | `Result<Ref<T>, _>` | Never (returns Err) |
| `try_borrow_mut()` | `Result<RefMut<T>, _>` | Never (returns Err) |

Violating the rules panics at runtime:

```rust
let r = RefCell::new(5);
let a = r.borrow();         // Immutable borrow
let b = r.borrow_mut();     // PANIC! Already borrowed immutably
```

The `Rc<RefCell<T>>` pattern combines shared ownership with interior mutability:

```rust
use std::rc::Rc;
use std::cell::RefCell;

let shared = Rc::new(RefCell::new(vec![1, 2, 3]));
let clone = Rc::clone(&shared);

clone.borrow_mut().push(4);
println!("{:?}", shared.borrow());  // [1, 2, 3, 4]
```

## Common Pitfalls
1. **Holding borrows too long** — A `Ref` or `RefMut` guard keeps the borrow active until it is dropped. Holding it across a call that also borrows the RefCell will panic.
2. **Using RefCell when Cell suffices** — If the type is Copy, Cell is simpler and has zero runtime overhead.

## Best Practices
1. **Keep borrow scopes short** — Drop the `Ref`/`RefMut` guard as soon as possible. Use blocks `{ }` to limit the scope.
2. **Use try_borrow for recoverable errors** — When a panic would be unacceptable, use `try_borrow()` and handle the error.

## Summary
- RefCell<T> enforces borrow rules at runtime with panics on violations.
- `borrow()` returns Ref<T> (shared), `borrow_mut()` returns RefMut<T> (exclusive).
- Rc<RefCell<T>> enables shared mutable state in single-threaded code.
- RefCell is not thread-safe — use Mutex or RwLock for concurrent access.

## Code Examples

**Mock with RefCell — the trait requires &self but the mock needs to record messages internally**

```rust
use std::cell::RefCell;

// Mock object pattern for testing
trait Messenger {
    fn send(&self, msg: &str);
}

struct MockMessenger {
    messages: RefCell<Vec<String>>,
}

impl MockMessenger {
    fn new() -> Self {
        MockMessenger {
            messages: RefCell::new(vec![]),
        }
    }
}

impl Messenger for MockMessenger {
    fn send(&self, msg: &str) {
        // &self but we can still record!
        self.messages.borrow_mut().push(msg.to_string());
    }
}
```


## Resources

- [RefCell Documentation](https://doc.rust-lang.org/std/cell/struct.RefCell.html) — Official API reference for RefCell

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*