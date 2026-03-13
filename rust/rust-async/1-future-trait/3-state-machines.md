---
source_course: "rust-async"
source_lesson: "rust-async-state-machines"
---

# Async State Machines

## Introduction

Every `async fn` compiles into an enum-based state machine. Each `.await` point becomes a variant, and local variables that live across await points are stored in the enum. Understanding this helps you reason about future sizes and performance.

## Key Concepts

An async function like this:

```rust
async fn example() -> i32 {
    let a = step_one().await;   // Suspension point 1
    let b = step_two(a).await;  // Suspension point 2
    a + b
}
```

Compiles roughly to:

```rust
enum ExampleFuture {
    State0 { step_one_fut: StepOneFuture },
    State1 { a: i32, step_two_fut: StepTwoFuture },
    Complete,
}
```

## Real World Context

This transformation is why Rust async has zero-cost overhead — there's no heap allocation for the state machine itself. It's a plain enum on the stack (or wherever the future is stored).

## Deep Dive

**State transitions on poll:**

```text
State0 --poll()--> Pending (step_one not done)
       --poll()--> State1 (step_one complete, start step_two)

State1 --poll()--> Pending (step_two not done)
       --poll()--> Ready(a + b)
```

**Memory layout matters.** The enum is as large as its biggest variant. Variables alive across `.await` are stored in the state machine:

```rust
async fn bloated() {
    let buf = vec![0u8; 1024]; // Stored across await!
    some_async().await;
    println!("{:?}", buf);
}
```

**Optimization: drop before await.**

```rust
async fn lean() {
    {
        let buf = vec![0u8; 1024];
        process(&buf);
    } // buf dropped here
    some_async().await; // Future is smaller
}
```

## Common Pitfalls

- Holding large buffers or collections across `.await` inflates the future size
- Deeply nested async calls produce deeply nested state machines
- `Box::pin` may be needed if future sizes are too large for the stack

## Best Practices

- Scope large temporaries so they drop before await points
- Use `std::mem::size_of_val(&my_future())` to inspect future sizes
- Consider `Box::pin` for recursive async functions

## Summary

Async functions become enum state machines. Each `.await` is a suspension point and a new variant. Variables alive across awaits inflate the future's size. Drop large values before awaiting to keep futures lean.

## Code Examples

**Inspecting future sizes to understand the state machine**

```rust
use std::mem::size_of_val;

async fn small() -> i32 { 42 }

async fn large() -> i32 {
    let buffer = [0u8; 1024];
    tokio::time::sleep(std::time::Duration::from_secs(1)).await;
    buffer.len() as i32
}

fn main() {
    println!("small future: {} bytes", size_of_val(&small()));
    println!("large future: {} bytes", size_of_val(&large()));
    // large is significantly bigger because buffer lives across await
}
```


## Resources

- [How Rust Async Works](https://rust-lang.github.io/async-book/01_getting_started/04_async_await_primer.html) — Async Book primer on how async/await compiles

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*