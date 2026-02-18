---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-rwlock-condvar"
---

# RwLock: Multiple Readers OR One Writer

```rust
use std::sync::RwLock;

let lock = RwLock::new(5);

// Multiple readers OK
{
    let r1 = lock.read().unwrap();
    let r2 = lock.read().unwrap();
    println!("{} {}", *r1, *r2);
}

// Writer needs exclusive access
{
    let mut w = lock.write().unwrap();
    *w += 1;
}
```

## When to Use RwLock vs Mutex

| Use Case | Choice |
|----------|--------|
| Many readers, few writers | RwLock |
| Mostly writes | Mutex |
| Short critical sections | Mutex |
| Read-heavy config | RwLock |

# Condvar: Wait for Conditions

```rust
use std::sync::{Arc, Mutex, Condvar};

let pair = Arc::new((Mutex::new(false), Condvar::new()));
let pair2 = Arc::clone(&pair);

// Waiter
thread::spawn(move || {
    let (lock, cvar) = &*pair2;
    let mut started = lock.lock().unwrap();
    while !*started {
        started = cvar.wait(started).unwrap();
    }
    println!("Worker: Started!");
});

// Signaler
thread::sleep(Duration::from_secs(1));
let (lock, cvar) = &*pair;
*lock.lock().unwrap() = true;
cvar.notify_one();
```

## Producer-Consumer with Condvar

```rust
use std::collections::VecDeque;

struct BoundedQueue<T> {
    data: Mutex<VecDeque<T>>,
    not_empty: Condvar,
    not_full: Condvar,
    capacity: usize,
}

impl<T> BoundedQueue<T> {
    fn push(&self, item: T) {
        let mut data = self.data.lock().unwrap();
        while data.len() >= self.capacity {
            data = self.not_full.wait(data).unwrap();
        }
        data.push_back(item);
        self.not_empty.notify_one();
    }
    
    fn pop(&self) -> T {
        let mut data = self.data.lock().unwrap();
        while data.is_empty() {
            data = self.not_empty.wait(data).unwrap();
        }
        let item = data.pop_front().unwrap();
        self.not_full.notify_one();
        item
    }
}
```

## Code Examples

**RwLock for config**

```rust
use std::sync::{Arc, RwLock};
use std::thread;

// Shared configuration
let config = Arc::new(RwLock::new(Config::default()));

// Many reader threads
for i in 0..10 {
    let cfg = Arc::clone(&config);
    thread::spawn(move || {
        loop {
            let config = cfg.read().unwrap();
            process_with_config(&config);
        }
    });
}

// Occasional config updates
let cfg = Arc::clone(&config);
thread::spawn(move || {
    loop {
        thread::sleep(Duration::from_secs(60));
        let mut config = cfg.write().unwrap();
        config.reload();
    }
});
```


---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*