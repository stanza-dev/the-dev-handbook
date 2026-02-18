---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-crossbeam-channels"
---

# Why Crossbeam?

The `crossbeam` crate provides superior channels:

- **MPMC**: Multiple consumers too!
- **Better performance**: Lock-free algorithms
- **Select**: Wait on multiple channels
- **Bounded**: Backpressure support

## Basic Usage

```rust
use crossbeam_channel::{unbounded, bounded};

// Unbounded (like mpsc::channel)
let (tx, rx) = unbounded();

// Bounded (like mpsc::sync_channel)
let (tx, rx) = bounded(10);

// MPMC: Multiple receivers!
let rx2 = rx.clone();
```

## The select! Macro

Wait on multiple channels:

```rust
use crossbeam_channel::{select, unbounded};

let (tx1, rx1) = unbounded();
let (tx2, rx2) = unbounded();

select! {
    recv(rx1) -> msg => println!("From rx1: {:?}", msg),
    recv(rx2) -> msg => println!("From rx2: {:?}", msg),
    default => println!("No messages ready"),
}
```

## Tick and After

```rust
use crossbeam_channel::{tick, after, select};
use std::time::Duration;

let ticker = tick(Duration::from_millis(100));
let timeout = after(Duration::from_secs(1));

loop {
    select! {
        recv(ticker) -> _ => println!("Tick!"),
        recv(timeout) -> _ => {
            println!("Timeout!");
            break;
        }
    }
}
```

## Bounded vs Unbounded

| | Bounded | Unbounded |
|---|---------|----------|
| `send` | Blocks when full | Never blocks |
| Memory | Limited | Can grow forever |
| Backpressure | Yes | No |
| Rendezvous | `bounded(0)` | N/A |

See [crossbeam-channel](https://docs.rs/crossbeam-channel/).

## Code Examples

**Work queue with MPMC**

```rust
use crossbeam_channel::{bounded, select, Receiver};
use std::thread;

fn worker(id: usize, jobs: Receiver<usize>) {
    for job in jobs {
        println!("Worker {id} processing job {job}");
        thread::sleep(std::time::Duration::from_millis(100));
    }
}

fn main() {
    let (tx, rx) = bounded(10);
    
    // Spawn workers (MPMC!)
    let handles: Vec<_> = (0..4)
        .map(|i| {
            let rx = rx.clone();
            thread::spawn(move || worker(i, rx))
        })
        .collect();
    
    // Send jobs
    for job in 0..20 {
        tx.send(job).unwrap();
    }
    drop(tx); // Close channel
    
    for h in handles {
        h.join().unwrap();
    }
}
```


## Resources

- [crossbeam-channel](https://docs.rs/crossbeam-channel/) — Crossbeam channel documentation

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*