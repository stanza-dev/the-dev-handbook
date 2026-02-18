---
source_course: "rust-async"
source_lesson: "rust-async-state-machines"
---

# How Async Compiles to State Machines

Every `async fn` becomes a state machine.

```rust
async fn example() -> i32 {
    let a = step_one().await;   // State 1
    let b = step_two(a).await;  // State 2
    a + b                       // State 3 (final)
}
```

Compiles roughly to:

```rust
enum ExampleFuture {
    State1 { step_one_future: StepOneFuture },
    State2 { a: i32, step_two_future: StepTwoFuture },
    Complete,
}
```

## State Transitions

Each `.await` point is a potential suspension point:

```
State1 ──poll()──► Pending (waiting for step_one)
       └──poll()──► State2 (step_one complete, start step_two)

State2 ──poll()──► Pending (waiting for step_two)  
       └──poll()──► Ready(a + b)
```

## Memory Layout

The state machine stores all local variables that live across await points:

```rust
async fn memory_example() {
    let large_buffer = vec![0u8; 1024]; // Stored in state machine
    some_async().await;                  // Suspension point
    println!("{:?}", large_buffer);     // Still needed after await
}
// Future size includes large_buffer!
```

## Optimization Tip

Drop large values before await points:

```rust
async fn optimized() {
    {
        let large = vec![0u8; 1024];
        process(&large);
    } // large dropped here
    some_async().await; // Future is smaller!
}
```

## Code Examples

**Future size inspection**

```rust
// Check Future size
use std::mem::size_of_val;

async fn small() -> i32 { 42 }

async fn large() -> i32 {
    let buffer = [0u8; 1024];
    tokio::time::sleep(std::time::Duration::from_secs(1)).await;
    buffer.len() as i32
}

fn main() {
    println!("small: {} bytes", size_of_val(&small()));
    println!("large: {} bytes", size_of_val(&large()));
}
```


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*