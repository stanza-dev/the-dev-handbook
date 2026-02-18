---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-thread-parking"
---

# Thread Control

## Thread Parking

Parking puts a thread to sleep until unparked:

```rust
use std::thread;
use std::time::Duration;

let parked = thread::spawn(|| {
    println!("Parking...");
    thread::park(); // Sleep until unparked
    println!("Unparked!");
});

thread::sleep(Duration::from_secs(1));
println!("Unparking the thread");
parked.thread().unpark(); // Wake up!
parked.join().unwrap();
```

## Parking with Timeout

```rust
thread::park_timeout(Duration::from_secs(5));
// Returns after 5 seconds OR when unparked
```

## Yielding

Give up the CPU to let other threads run:

```rust
for i in 0..1000 {
    do_work(i);
    if i % 100 == 0 {
        thread::yield_now(); // Let other threads run
    }
}
```

## Spin Loops

For very short waits (nanoseconds):

```rust
while !condition.load(Ordering::Acquire) {
    std::hint::spin_loop(); // CPU hint: we're spinning
}
```

`spin_loop()` tells the CPU we're waiting, which can save power and improve performance on hyper-threaded CPUs.

## Code Examples

**Thread parking notification**

```rust
use std::sync::atomic::{AtomicBool, Ordering};
use std::sync::Arc;
use std::thread;

// Simple notification with parking
fn main() {
    let ready = Arc::new(AtomicBool::new(false));
    let ready_clone = Arc::clone(&ready);
    
    let worker = thread::spawn(move || {
        // Wait for signal
        while !ready_clone.load(Ordering::Acquire) {
            thread::park();
        }
        println!("Worker: Got signal, starting work!");
    });
    
    thread::sleep(std::time::Duration::from_millis(100));
    println!("Main: Sending signal");
    ready.store(true, Ordering::Release);
    worker.thread().unpark();
    
    worker.join().unwrap();
}
```


---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*