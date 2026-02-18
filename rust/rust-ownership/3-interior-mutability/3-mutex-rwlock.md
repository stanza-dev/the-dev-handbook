---
source_course: "rust-ownership"
source_lesson: "rust-ownership-mutex-rwlock"
---

# Thread-Safe Interior Mutability

`Cell` and `RefCell` are `!Sync` - they can't be shared between threads. For threads, use `Mutex` or `RwLock`.

## Mutex<T>

```rust
use std::sync::Mutex;

let m = Mutex::new(5);

{
    let mut guard = m.lock().unwrap();
    *guard = 10;
}  // Lock released when guard drops

println!("{:?}", m.lock().unwrap());  // 10
```

## Arc<Mutex<T>> Pattern

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    handles.push(thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    }));
}

for handle in handles {
    handle.join().unwrap();
}

println!("Count: {}", *counter.lock().unwrap());  // 10
```

## RwLock<T>: Multiple Readers

```rust
use std::sync::RwLock;

let lock = RwLock::new(5);

// Multiple readers OK
let r1 = lock.read().unwrap();
let r2 = lock.read().unwrap();
drop(r1); drop(r2);

// Writer needs exclusive access
let mut w = lock.write().unwrap();
*w += 1;
```

## Comparison Table

| Type | Thread-safe | Multiple readers | Use case |
|------|-------------|------------------|----------|
| Cell | No | N/A | Copy types, single thread |
| RefCell | No | Yes (runtime) | Complex borrows, single thread |
| Mutex | Yes | No | Exclusive access |
| RwLock | Yes | Yes | Read-heavy workloads |

## Code Examples

**RwLock for config**

```rust
use std::sync::{Arc, RwLock};
use std::thread;

// Read-heavy configuration
let config = Arc::new(RwLock::new(Config::default()));

// Many readers
for i in 0..10 {
    let config = Arc::clone(&config);
    thread::spawn(move || {
        let cfg = config.read().unwrap();
        println!("Reader {i}: {}", cfg.setting);
    });
}

// Occasional writer
{
    let mut cfg = config.write().unwrap();
    cfg.setting = "updated".to_string();
}
```


---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*