---
source_course: "rust-performance"
source_lesson: "rust-perf-criterion"
---

# Criterion.rs: Statistical Benchmarking

## Introduction

Criterion.rs is the gold-standard benchmarking framework for Rust. It provides statistically rigorous measurements, regression detection, and HTML reports — all on stable Rust. As of Rust 1.88, the built-in `#[bench]` attribute is a hard error on stable (fully de-stabilized), making Criterion (or Divan) the only benchmarking options without nightly.

## Key Concepts

Criterion.rs has moved to the `criterion-rs` GitHub organization. The old `bheisler/criterion.rs` repository is unmaintained. Always use v0.8+ from the new org.

**Setup:**

```toml
# Cargo.toml
[dev-dependencies]
criterion = "0.8"

[[bench]]
name = "my_benchmark"
harness = false
```

**Basic benchmark:**

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 | 1 => 1,
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

## Real World Context

Every performance-critical Rust library uses Criterion for continuous benchmarking. CI pipelines compare benchmarks against baselines to catch regressions before they ship.

## Deep Dive

### Why `black_box`?

`black_box` is an optimization barrier. Without it the compiler may compute results at compile time, see that the result is unused, or inline and eliminate entire call trees:

```rust
b.iter(|| {
    let result = expensive_function(black_box(input));
    black_box(result); // Prevent dead-code elimination
});
```

### Parameterized Benchmarks

```rust
use criterion::BenchmarkId;

fn bench_sorting(c: &mut Criterion) {
    let mut group = c.benchmark_group("Sorting");
    for size in [100, 1_000, 10_000] {
        group.bench_with_input(
            BenchmarkId::new("sort_unstable", size),
            &size,
            |b, &size| {
                let mut data: Vec<i32> = (0..size).rev().collect();
                b.iter(|| {
                    data.sort_unstable();
                });
            },
        );
    }
    group.finish();
}
```

### Running & Comparing

```bash
cargo bench
cargo bench -- --save-baseline main
# After changes:
cargo bench -- --baseline main
```

Criterion generates HTML reports in `target/criterion/`.

## Common Pitfalls

- Using the old `criterion = "0.5"` from the unmaintained repo. Always use `criterion = "0.8"` from the `criterion-rs` org.
- Forgetting `harness = false` in `Cargo.toml` — your benchmarks will not compile.
- Attempting `#[bench]` on stable Rust 1.88+ — it is a hard error now.
- Benchmarking debug builds — always run `cargo bench` which uses the `bench` profile (release-level optimization).

## Best Practices

- Wrap both inputs and outputs with `black_box`.
- Use `BenchmarkGroup` for comparing implementations at different input sizes.
- Save baselines in CI to detect regressions automatically.
- Keep benchmarks focused on a single operation.

## Summary

Criterion.rs v0.8 is the standard for Rust benchmarking on stable. It uses statistical analysis to produce reliable measurements, detects regressions, and generates visual reports. With `#[bench]` removed from stable, Criterion is essential for any performance work.

## Code Examples

**Parameterized Criterion benchmark comparing sort vs sort_unstable across input sizes**

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion, BenchmarkId};

fn bench_sorting(c: &mut Criterion) {
    let mut group = c.benchmark_group("Sorting");

    for size in [100, 1_000, 10_000] {
        group.bench_with_input(
            BenchmarkId::new("Vec::sort", size),
            &size,
            |b, &size| {
                let mut data: Vec<i32> = (0..size).rev().collect();
                b.iter(|| {
                    data.sort();
                    black_box(&data);
                });
            },
        );

        group.bench_with_input(
            BenchmarkId::new("Vec::sort_unstable", size),
            &size,
            |b, &size| {
                let mut data: Vec<i32> = (0..size).rev().collect();
                b.iter(|| {
                    data.sort_unstable();
                    black_box(&data);
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

- [Criterion.rs User Guide](https://bheisler.github.io/criterion.rs/book/) — Complete guide to Criterion.rs benchmarking
- [The Rust Performance Book](https://nnethercote.github.io/perf-book/) — Comprehensive guide to Rust performance optimization

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*