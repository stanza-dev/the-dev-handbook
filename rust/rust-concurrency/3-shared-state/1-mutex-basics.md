---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-mutex-basics"
---

# Mutex: One Thread at a Time

A Mutex ensures only one thread can access the data at a time.

```rust
use std::sync::Mutex;

let m = Mutex::new(5);

{
    let mut guard = m.lock().unwrap();
    *guard = 6;
} // Lock released when guard drops

println!("{:?}", m.lock().unwrap()); // 6
```

## Arc<Mutex<T>> Pattern

Share across threads:

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}

println!("Result: {}", *counter.lock().unwrap()); // 10
```

## try_lock: Non-Blocking

```rust
match m.try_lock() {
    Ok(mut guard) => {
        *guard += 1;
    }
    Err(_) => {
        println!("Lock is held by another thread");
    }
}
```

## Poisoning

If a thread panics while holding a lock, the Mutex becomes "poisoned":

```rust
match m.lock() {
    Ok(guard) => { /* use guard */ }
    Err(poisoned) => {
        // Recover the data despite the panic
        let guard = poisoned.into_inner();
        println!("Recovered: {:?}", *guard);
    }
}
```

See [Shared-State Concurrency](https://doc.rust-lang.org/book/ch16-03-shared-state.html).

## Code Examples

**Thread-safe counter**

```rust
use std::sync::{Arc, Mutex};
use std::thread;

// Thread-safe counter
struct Counter {
    value: Mutex<usize>,
}

impl Counter {
    fn new() -> Self {
        Counter { value: Mutex::new(0) }
    }
    
    fn increment(&self) {
        let mut val = self.value.lock().unwrap();
        *val += 1;
    }
    
    fn get(&self) -> usize {
        *self.value.lock().unwrap()
    }
}

let counter = Arc::new(Counter::new());

thread::scope(|s| {
    for _ in 0..10 {
        let c = Arc::clone(&counter);
        s.spawn(move || {
            for _ in 0..100 {
                c.increment();
            }
        });
    }
});

println!("Final: {}", counter.get()); // 1000
```


---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*