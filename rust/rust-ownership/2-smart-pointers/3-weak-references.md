---
source_course: "rust-ownership"
source_lesson: "rust-ownership-weak-references"
---

# Weak References & Reference Cycles

## Introduction
Reference counting has a well-known weakness: cycles. If two Rc values point to each other, their counts never reach zero and the memory leaks. Weak references solve this by providing non-owning references that do not prevent deallocation.

## Key Concepts
- **Reference cycle**: A situation where two or more Rc values form a loop, preventing any of them from reaching a zero strong count.
- **Weak<T>**: A non-owning reference created from an Rc. It does not increment the strong count and must be upgraded to an Rc before use.
- **Upgrade**: Converting a Weak to an `Option<Rc<T>>` — returns `None` if the data has been dropped.

## Real World Context
Tree structures where children need a reference to their parent are the classic use case. Without Weak, a parent owning children (via Rc) and children pointing back to parents (also via Rc) creates a cycle. Observer patterns similarly benefit from Weak when observers should not prevent the subject from being freed.

## Deep Dive
The reference cycle problem occurs when Rc values form a loop:

```rust
use std::rc::Rc;
use std::cell::RefCell;

struct Node {
    next: Option<Rc<RefCell<Node>>>,
}

let a = Rc::new(RefCell::new(Node { next: None }));
let b = Rc::new(RefCell::new(Node { next: Some(Rc::clone(&a)) }));
a.borrow_mut().next = Some(Rc::clone(&b));
// Memory leak! a -> b -> a, counts never reach 0
```

The solution is to use Weak for the "back" reference:

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct Node {
    parent: RefCell<Weak<Node>>,      // Weak: does not own
    children: RefCell<Vec<Rc<Node>>>,  // Strong: owns children
}

let parent = Rc::new(Node {
    parent: RefCell::new(Weak::new()),
    children: RefCell::new(vec![]),
});

let child = Rc::new(Node {
    parent: RefCell::new(Rc::downgrade(&parent)),
    children: RefCell::new(vec![]),
});

parent.children.borrow_mut().push(Rc::clone(&child));
```

To use a Weak reference, you must upgrade it first, which returns `None` if the data was already dropped:

```rust
let weak: Weak<Node> = Rc::downgrade(&strong);

match weak.upgrade() {
    Some(rc) => println!("Still alive!"),
    None => println!("Already dropped"),
}
```

## Common Pitfalls
1. **Forgetting to upgrade Weak before use** — Weak does not implement Deref. You must call `upgrade()` and handle the `None` case.
2. **Using Rc where Weak is needed** — Any back-reference in a parent-child relationship should use Weak to prevent cycles.

## Best Practices
1. **Owner holds Strong, dependent holds Weak** — In a tree, parents own children (Rc), children reference parents (Weak). This ensures predictable drop order.
2. **Check strong and weak counts during debugging** — `Rc::strong_count` and `Rc::weak_count` help diagnose leaks during development.

## Summary
- Reference cycles prevent Rc from freeing memory.
- Weak<T> provides non-owning references that break cycles.
- Upgrading a Weak returns Option<Rc<T>> — None if the data was dropped.
- Use Strong for ownership direction, Weak for back-references.

## Code Examples

**Tree with weak parent refs — children hold Weak references to parents, preventing reference cycles**

```rust
// Tree with parent references
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct TreeNode {
    value: i32,
    parent: RefCell<Weak<TreeNode>>,
    children: RefCell<Vec<Rc<TreeNode>>>,
}

impl TreeNode {
    fn new(value: i32) -> Rc<Self> {
        Rc::new(Self {
            value,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![]),
        })
    }
    
    fn add_child(parent: &Rc<Self>, child: Rc<Self>) {
        *child.parent.borrow_mut() = Rc::downgrade(parent);
        parent.children.borrow_mut().push(child);
    }
}
```


## Resources

- [Weak<T> Documentation](https://doc.rust-lang.org/std/rc/struct.Weak.html) — Official API reference for Weak

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*