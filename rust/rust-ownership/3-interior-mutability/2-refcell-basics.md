---
source_course: "rust-ownership"
source_lesson: "rust-ownership-refcell-basics"
---

# RefCell<T>: Borrow Rules at Runtime

`RefCell<T>` enforces borrowing rules at **runtime** instead of compile time.

```rust
use std::cell::RefCell;

let x = RefCell::new(5);

{
    let mut borrow = x.borrow_mut();
    *borrow += 1;
}  // Mutable borrow ends

println!("{}", x.borrow());  // 6
```

## The Methods

| Method | Returns | Panics if |
|--------|---------|----------|
| `borrow()` | `Ref<T>` | Already mutably borrowed |
| `borrow_mut()` | `RefMut<T>` | Already borrowed (any) |
| `try_borrow()` | `Result<Ref<T>, _>` | Never |
| `try_borrow_mut()` | `Result<RefMut<T>, _>` | Never |

## Runtime Panics!

```rust
let r = RefCell::new(5);
let a = r.borrow();
let b = r.borrow_mut();  // PANIC! Already borrowed immutably
```

## Rc<RefCell<T>> Pattern

Combine for multiple owners with mutation:

```rust
use std::rc::Rc;
use std::cell::RefCell;

let shared = Rc::new(RefCell::new(vec![1, 2, 3]));
let clone = Rc::clone(&shared);

clone.borrow_mut().push(4);
println!("{:?}", shared.borrow());  // [1, 2, 3, 4]
```

## When to Use RefCell

1. When you're sure the borrow rules are followed, but compiler can't prove it
2. Mock objects in tests
3. Graph structures with shared mutable nodes

See [RefCell](https://doc.rust-lang.org/std/cell/struct.RefCell.html).

## Code Examples

**Mock with RefCell**

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

#[test]
fn test_sends_message() {
    let mock = MockMessenger::new();
    mock.send("hello");
    assert_eq!(mock.messages.borrow().len(), 1);
}
```


---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*