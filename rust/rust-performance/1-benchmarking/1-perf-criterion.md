---
source_course: "rust-performance"
source_lesson: "rust-perf-criterion"
---

# Criterion.rs

The gold standard for Rust benchmarking. Works on stable Rust.

## Setup

```toml
# Cargo.toml
[dev-dependencies]
criterion = "0.5"

[[bench]]
name = "my_benchmark"
harness = false
```

## Basic Benchmark

```rust
// benches/my_benchmark.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 1,
        1 => 1,
        n => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

fn criterion_benchmark(c: &mut Criterion) {
    c.bench_function("fib 20", |b| {
        b.iter(|| fibonacci(black_box(20)))
    });
}

criterion_group!(benches, criterion_benchmark);
criterion_main!(benches);
```

## Why black_box?

`black_box` prevents the compiler from optimizing away your code:

```rust
// Without black_box, compiler might:
// 1. Compute result at compile time
// 2. See result is unused and remove call entirely

b.iter(|| {
    let result = expensive_function(black_box(input));
    black_box(result)  // Prevent optimizing away
});
```

## Running Benchmarks

```bash
cargo bench

# Compare to baseline
cargo bench -- --save-baseline main
cargo bench -- --baseline main
```

Criterion generates HTML reports in `target/criterion/`.

See [Criterion.rs](https://bheisler.github.io/criterion.rs/book/).

## Code Examples

**Parameterized benchmarks**

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion, BenchmarkId};

fn bench_sorting(c: &mut Criterion) {
    let mut group = c.benchmark_group("Sorting");
    
    for size in [100, 1000, 10000].iter() {
        group.bench_with_input(
            BenchmarkId::new("Vec::sort", size),
            size,
            |b, &size| {
                let mut data: Vec<i32> = (0..size).rev().collect();
                b.iter(|| {
                    data.shuffle(&mut rand::thread_rng());
                    data.sort();
                });
            },
        );
        
        group.bench_with_input(
            BenchmarkId::new("Vec::sort_unstable", size),
            size,
            |b, &size| {
                let mut data: Vec<i32> = (0..size).rev().collect();
                b.iter(|| {
                    data.shuffle(&mut rand::thread_rng());
                    data.sort_unstable();
                });
            },
        );
    }
    group.finish();
}

criterion_group!(benches, bench_sorting);
criterion_main!(benches);
```


## Resources

- [Criterion.rs Book](https://bheisler.github.io/criterion.rs/book/) — Complete guide to Criterion

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*