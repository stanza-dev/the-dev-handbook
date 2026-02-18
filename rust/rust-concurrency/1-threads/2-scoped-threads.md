---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-scoped-threads"
---

# The Problem with spawn

Standard `spawn` requires `'static` because threads might outlive the function:

```rust
let data = vec![1, 2, 3];
let slice = &data[..];

thread::spawn(|| {
    println!("{slice:?}"); // Error! slice isn't 'static
});
```

# Scoped Threads (Rust 1.63+)

`thread::scope` guarantees threads finish before the scope ends:

```rust
let mut data = vec![1, 2, 3];

thread::scope(|s| {
    s.spawn(|| {
        println!("Thread 1: {data:?}"); // Borrow OK!
    });
    
    s.spawn(|| {
        println!("Thread 2: {data:?}");
    });
}); // All threads joined automatically here

// data is still accessible
println!("After scope: {data:?}");
```

## Mutable Borrows with Scoped Threads

```rust
let mut data = vec![1, 2, 3, 4, 5, 6];
let (left, right) = data.split_at_mut(3);

thread::scope(|s| {
    s.spawn(|| {
        for x in left.iter_mut() {
            *x *= 2;
        }
    });
    
    s.spawn(|| {
        for x in right.iter_mut() {
            *x *= 3;
        }
    });
});

println!("{data:?}"); // [2, 4, 6, 12, 15, 18]
```

## Parallel Chunk Processing

```rust
fn parallel_process(data: &mut [i32]) {
    let chunk_size = data.len() / 4;
    
    thread::scope(|s| {
        for chunk in data.chunks_mut(chunk_size) {
            s.spawn(|| {
                for x in chunk {
                    *x = expensive_computation(*x);
                }
            });
        }
    });
}
```

See [Scoped Threads](https://doc.rust-lang.org/std/thread/fn.scope.html).

## Code Examples

**Parallel sum with scoped threads**

```rust
use std::thread;

fn parallel_sum(numbers: &[i32]) -> i32 {
    let mid = numbers.len() / 2;
    let (left, right) = numbers.split_at(mid);
    
    let mut left_sum = 0;
    let mut right_sum = 0;
    
    thread::scope(|s| {
        s.spawn(|| {
            left_sum = left.iter().sum();
        });
        s.spawn(|| {
            right_sum = right.iter().sum();
        });
    });
    
    left_sum + right_sum
}

let numbers: Vec<i32> = (1..=1000).collect();
println!("Sum: {}", parallel_sum(&numbers));
```


---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*