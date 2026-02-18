---
source_course: "rust-ownership"
source_lesson: "rust-ownership-rc-arc"
---

# Reference Counting: Multiple Owners

Sometimes data needs multiple owners. Rc (Reference Counted) enables this.

## Rc<T>: Single-Threaded

```rust
use std::rc::Rc;

let a = Rc::new(String::from("hello"));
let b = Rc::clone(&a);  // Increments count, doesn't copy data
let c = Rc::clone(&a);

println!("Count: {}", Rc::strong_count(&a));  // 3
```

Data is dropped when the last Rc is dropped.

## Rc Use Case: Shared Graph Nodes

```rust
use std::rc::Rc;

struct Node {
    value: i32,
    children: Vec<Rc<Node>>,
}

let shared = Rc::new(Node { value: 1, children: vec![] });
let parent1 = Node { value: 2, children: vec![Rc::clone(&shared)] };
let parent2 = Node { value: 3, children: vec![Rc::clone(&shared)] };
// shared has two parents!
```

## Arc<T>: Thread-Safe

Arc (Atomic Reference Counted) is the thread-safe version:

```rust
use std::sync::Arc;
use std::thread;

let data = Arc::new(vec![1, 2, 3]);

let handles: Vec<_> = (0..3).map(|_| {
    let data = Arc::clone(&data);
    thread::spawn(move || {
        println!("{:?}", data);
    })
}).collect();
```

## Rc vs Arc

| | Rc | Arc |
|---|---|---|
| Thread-safe | No | Yes |
| Performance | Faster | Atomic operations |
| Send + Sync | Neither | Both |

⚠️ **Neither allows mutation!** Use with RefCell or Mutex.

See [Rc](https://doc.rust-lang.org/std/rc/struct.Rc.html) and [Arc](https://doc.rust-lang.org/std/sync/struct.Arc.html).

## Code Examples

**Sharing config with Arc**

```rust
use std::sync::Arc;
use std::thread;

// Sharing read-only config across threads
let config = Arc::new(Config::load());

let handles: Vec<_> = (0..4).map(|i| {
    let config = Arc::clone(&config);
    thread::spawn(move || {
        println!("Worker {i}: {}", config.setting);
    })
}).collect();

for h in handles {
    h.join().unwrap();
}
```


---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*