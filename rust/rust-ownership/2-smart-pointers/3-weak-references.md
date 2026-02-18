---
source_course: "rust-ownership"
source_lesson: "rust-ownership-weak-references"
---

# Weak<T>: Non-Owning References

Weak references don't prevent deallocation - they break reference cycles.

## The Reference Cycle Problem

```rust
use std::rc::Rc;
use std::cell::RefCell;

struct Node {
    next: Option<Rc<RefCell<Node>>>,
}

let a = Rc::new(RefCell::new(Node { next: None }));
let b = Rc::new(RefCell::new(Node { next: Some(Rc::clone(&a)) }));
a.borrow_mut().next = Some(Rc::clone(&b));
// Memory leak! a → b → a, count never reaches 0
```

## Solution: Weak References

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct Node {
    parent: RefCell<Weak<Node>>,  // Weak: doesn't own
    children: RefCell<Vec<Rc<Node>>>,  // Strong: owns
}

let parent = Rc::new(Node {
    parent: RefCell::new(Weak::new()),
    children: RefCell::new(vec![]),
});

let child = Rc::new(Node {
    parent: RefCell::new(Rc::downgrade(&parent)),  // Weak ref
    children: RefCell::new(vec![]),
});

parent.children.borrow_mut().push(Rc::clone(&child));
```

## Using Weak References

```rust
let weak: Weak<Node> = Rc::downgrade(&strong);

// Must upgrade to use (might be None if dropped)
match weak.upgrade() {
    Some(rc) => println!("Still alive!"),
    None => println!("Already dropped"),
}
```

## Strong vs Weak Counts

```rust
println!("Strong: {}", Rc::strong_count(&rc));
println!("Weak: {}", Rc::weak_count(&rc));
// Dropped when strong_count == 0
// Weak refs then return None on upgrade()
```

## Code Examples

**Tree with weak parent refs**

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


---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*