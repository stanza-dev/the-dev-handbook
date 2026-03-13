---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-thread-parking"
---

# Thread Parking & Yielding

## Introduction

Sometimes threads need to pause and wait for a signal, or politely give up the CPU so other threads can make progress. Rust provides three mechanisms for this: parking (sleep until woken), yielding (voluntarily reschedule), and spin-loop hints (busy-wait efficiently). Each serves a different latency-vs-efficiency trade-off.

## Key Concepts

- **Thread Parking**: Puts a thread to sleep until another thread calls `unpark()` on it. The thread consumes no CPU while parked.
- **Yielding**: `thread::yield_now()` tells the OS scheduler that this thread is willing to give up its time slice. The thread remains runnable and may be immediately rescheduled.
- **Spin Loop**: `std::hint::spin_loop()` is a CPU hint for busy-waiting. The processor can enter a low-power state during the spin, improving performance on hyper-threaded cores.
- **Park Token**: Parking uses a single-bit token internally. An `unpark` call before a `park` call "consumes" the token immediately, so the next `park` returns instantly instead of blocking.

## Real World Context

Thread parking is useful for building lightweight notification systems — a background thread parks itself and a producer unparks it when new work arrives. Yielding helps cooperative multitasking in tight loops that should not starve other threads. Spin loops are reserved for lock-free algorithms where you expect the wait to last only nanoseconds (e.g., spinlocks, compare-and-swap retry loops).

## Deep Dive

Parking puts the current thread to sleep until another thread explicitly wakes it up.

```rust
use std::thread;
use std::time::Duration;

let parked = thread::spawn(|| {
    println!("going to sleep...");
    thread::park(); // sleep until unparked
    println!("woke up!");
});

thread::sleep(Duration::from_millis(500));
println!("waking the thread");
parked.thread().unpark(); // wake it up
parked.join().unwrap();
```

Parking supports a timeout variant so you do not wait forever.

```rust
// Park for at most 5 seconds
thread::park_timeout(Duration::from_secs(5));
// Execution resumes here after being unparked OR after the timeout
```

An important subtlety is that `unpark` sets a token. If `unpark` is called *before* `park`, the next `park` call returns immediately. This avoids a common race condition, but it also means you should always re-check your condition in a loop.

```rust
use std::sync::atomic::{AtomicBool, Ordering};
use std::sync::Arc;

let flag = Arc::new(AtomicBool::new(false));
let flag_clone = Arc::clone(&flag);

let worker = thread::spawn(move || {
    while !flag_clone.load(Ordering::Acquire) {
        thread::park(); // spurious wakeups are possible
    }
    println!("flag is set, starting work");
});

thread::sleep(Duration::from_millis(100));
flag.store(true, Ordering::Release);
worker.thread().unpark();
worker.join().unwrap();
```

Yielding is a lighter-weight mechanism. It does not put the thread to sleep — it simply hints to the OS that now would be a good time to schedule another thread.

```rust
for i in 0..10_000 {
    do_work(i);
    if i % 100 == 0 {
        thread::yield_now(); // let other threads run
    }
}
```

For the shortest waits — nanosecond-scale polling in lock-free algorithms — use `std::hint::spin_loop()`. This emits a CPU instruction (`PAUSE` on x86, `YIELD` on ARM) that reduces power consumption and avoids starving hyper-threaded sibling cores.

```rust
use std::sync::atomic::{AtomicBool, Ordering};

let ready = AtomicBool::new(false);

// Spin until ready (only appropriate for very short waits)
while !ready.load(Ordering::Acquire) {
    std::hint::spin_loop();
}
```

## Common Pitfalls

1. **Not looping around `park`** — `park` can return spuriously (without an `unpark` call). Always recheck your condition in a `while` loop, not with a single `if`.
2. **Using spin loops for long waits** — `spin_loop()` keeps the CPU busy. If the wait might exceed a few microseconds, use parking, a `Condvar`, or a channel instead.

## Best Practices

1. **Prefer higher-level primitives** — Channels and `Condvar` are usually clearer and less error-prone than raw parking. Reserve parking for building your own synchronization primitives.
2. **Match the mechanism to the wait duration** — spin loop for nanoseconds, parking for milliseconds, and channels or condition variables for general-purpose signaling.

## Summary

- `thread::park()` sleeps the current thread until `unpark()` is called; always re-check conditions in a loop.
- `thread::yield_now()` voluntarily gives up the CPU time slice without sleeping.
- `std::hint::spin_loop()` is a busy-wait CPU hint for nanosecond-scale polling in lock-free code.
- The park/unpark token mechanism prevents lost wakeups but does not prevent spurious ones.

## Code Examples

**Notification pattern using park/unpark with an atomic flag for correctness**

```rust
use std::sync::atomic::{AtomicBool, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let ready = Arc::new(AtomicBool::new(false));
    let ready_clone = Arc::clone(&ready);

    let worker = thread::spawn(move || {
        // Wait for signal using park loop
        while !ready_clone.load(Ordering::Acquire) {
            thread::park();
        }
        println!("Worker: signal received, starting work!");
    });

    thread::sleep(std::time::Duration::from_millis(100));
    println!("Main: sending signal");
    ready.store(true, Ordering::Release);
    worker.thread().unpark();

    worker.join().unwrap();
}
```


## Resources

- [std::thread::park](https://doc.rust-lang.org/std/thread/fn.park.html) — Standard library reference for thread parking
- [std::hint::spin_loop](https://doc.rust-lang.org/std/hint/fn.spin_loop.html) — CPU hint for busy-wait spin loops

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*