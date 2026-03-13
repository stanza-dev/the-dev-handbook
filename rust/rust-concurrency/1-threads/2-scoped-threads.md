---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-scoped-threads"
---

# Scoped Threads

## Introduction

`thread::spawn` requires that everything the closure captures lives for `'static`, which forces you to clone or move data even when a short-lived borrow would be safe. Scoped threads, stabilized in Rust 1.63, solve this by guaranteeing that every thread spawned inside a scope is joined before the scope exits — so local stack variables can be safely borrowed.

## Key Concepts

- **Scope**: Created by `thread::scope(|s| { ... })`. The closure receives a `&Scope` handle used to spawn threads. The scope blocks until all spawned threads finish.
- **Non-`'static` Borrows**: Because the scope guarantees join-before-drop, closures passed to `s.spawn` may borrow local variables without `'static`.
- **Mutable Splitting**: You can give disjoint mutable slices to different threads using `split_at_mut`, enabling safe parallel mutation.

## Real World Context

Scoped threads shine in data-parallel algorithms where you split a large buffer into chunks and process them in parallel. Think parallel sorting, image processing by scanline, or matrix operations — any time you want to borrow and mutate owned data across threads without the overhead of `Arc`.

## Deep Dive

The core issue with `thread::spawn` is the `'static` bound. Spawned threads might outlive the function that created them, so the compiler cannot allow borrows of local variables.

```rust
use std::thread;

let data = vec![1, 2, 3];
let slice = &data[..];

// This fails to compile:
// thread::spawn(|| {
//     println!("{slice:?}"); // Error: `slice` is not 'static
// });
```

`thread::scope` fixes this by creating a bounded lifetime. All threads spawned inside the scope are automatically joined when the scope closure returns, so the compiler can prove that `data` outlives every thread.

```rust
use std::thread;

let data = vec![1, 2, 3];

thread::scope(|s| {
    s.spawn(|| {
        println!("thread 1: {data:?}"); // shared borrow — OK!
    });

    s.spawn(|| {
        println!("thread 2: {data:?}"); // another shared borrow — OK!
    });
}); // both threads joined here, before `data` is dropped

println!("after scope: {data:?}");
```

Scoped threads also enable safe parallel mutation through disjoint mutable borrows. Use `split_at_mut` or `chunks_mut` to give each thread a unique, non-overlapping slice.

```rust
use std::thread;

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

For larger workloads, `chunks_mut` lets you split data into arbitrarily many pieces and process them in parallel.

```rust
use std::thread;

fn parallel_process(data: &mut [i32]) {
    let num_threads = thread::available_parallelism()
        .map(|n| n.get())
        .unwrap_or(4);
    let chunk_size = (data.len() + num_threads - 1) / num_threads;

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

Scoped thread handles also return values, just like regular `JoinHandle`s.

```rust
use std::thread;

let numbers: Vec<i32> = (1..=1_000_000).collect();
let mid = numbers.len() / 2;
let (left, right) = numbers.split_at(mid);

let total = thread::scope(|s| {
    let left_handle = s.spawn(|| left.iter().sum::<i32>());
    let right_handle = s.spawn(|| right.iter().sum::<i32>());

    left_handle.join().unwrap() + right_handle.join().unwrap()
});

println!("total: {total}");
```

## Common Pitfalls

1. **Trying to use `&mut` without splitting** — Two threads cannot hold `&mut` references to the same data. Always split with `split_at_mut` or `chunks_mut` to create disjoint borrows.
2. **Panics inside scoped threads** — If a scoped thread panics, the scope still joins all other threads before propagating the panic. However, you may get confusing double-panic behavior if multiple threads panic.

## Best Practices

1. **Prefer scoped threads over `Arc` for short-lived parallelism** — When the parallel work is bounded and you already own the data, scoped threads avoid the overhead and complexity of reference counting.
2. **Size chunks to available parallelism** — Use `thread::available_parallelism()` to avoid spawning more threads than CPU cores, which adds context-switch overhead without improving throughput.

## Summary

- `thread::scope` guarantees all spawned threads join before the scope returns, enabling non-`'static` borrows.
- Disjoint mutable slices via `split_at_mut` or `chunks_mut` allow safe parallel mutation.
- Scoped threads eliminate the need for `Arc` in fork-join parallelism patterns.
- Scoped thread handles support return values via `join()`, just like regular threads.

## Code Examples

**Fork-join parallel sum using scoped threads to borrow stack-local data**

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


## Resources

- [std::thread::scope](https://doc.rust-lang.org/std/thread/fn.scope.html) — Standard library reference for scoped threads
- [Rust 1.63 Release Notes](https://blog.rust-lang.org/2022/08/11/Rust-1.63.0.html) — Release announcement that stabilized scoped threads

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*