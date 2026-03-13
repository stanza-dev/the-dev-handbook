---
source_course: "rust-performance"
source_lesson: "rust-perf-interpreting-results"
---

# Interpreting Benchmark Results

## Introduction

Running benchmarks is easy. Understanding what they tell you requires statistical literacy. Criterion provides rich statistical output — this lesson teaches you to read it.

## Key Concepts

### Criterion Output

A typical Criterion report looks like:

```
fib 20     time:   [26.029 µs 26.251 µs 26.505 µs]
           change: [-1.5430% -0.3426% +0.8682%] (p = 0.58 > 0.05)
           No change in performance detected.
```

- **time: [lower bound, estimate, upper bound]** — the 95% confidence interval for the mean execution time.
- **change** — percentage change from the last baseline.
- **p-value** — statistical significance. p < 0.05 means the change is likely real.

### Statistical Significance

Criterion runs your benchmark thousands of times and applies statistical tests. A reported regression of -2% with p = 0.60 is noise. A regression of -2% with p = 0.001 is real.

### Throughput Benchmarks

```rust
group.throughput(Throughput::Bytes(data.len() as u64));
group.bench_function("compress", |b| {
    b.iter(|| compress(black_box(&data)))
});
```

This reports both time and throughput (e.g., 450 MiB/s).

## Real World Context

CI pipelines save Criterion baselines and compare on each pull request. A statistically significant regression blocks the merge.

## Deep Dive

### Divan: An Alternative

Divan (v0.1.21) is a newer benchmarking framework using attribute macros instead of Criterion's macro DSL:

```rust
// Cargo.toml: divan = "0.1"

fn main() {
    divan::main();
}

#[divan::bench]
fn fibonacci() -> u64 {
    fn fib(n: u64) -> u64 {
        match n {
            0 | 1 => 1,
            n => fib(n - 1) + fib(n - 2),
        }
    }
    fib(divan::black_box(20))
}
```

Divan produces clean terminal output and is simpler to set up. Criterion remains more feature-rich (HTML reports, regression detection, baselines).

### Comparing Baselines

```bash
git checkout main
cargo bench -- --save-baseline main
git checkout feature-branch
cargo bench -- --baseline main
```

Criterion will report whether each benchmark improved or regressed relative to the baseline.

## Common Pitfalls

- Running benchmarks on a laptop with CPU frequency scaling — results vary wildly. Disable turbo boost or use a dedicated machine.
- Comparing single-run numbers without statistical analysis — always use confidence intervals.
- Ignoring the p-value and reacting to every reported change.

## Best Practices

- Always run benchmarks multiple times; trust Criterion's statistical output.
- Use `--save-baseline` in CI to track performance over time.
- Consider Divan for simpler benchmarks where HTML reports are not needed.
- Set throughput on I/O benchmarks so results are reported in bytes/sec.

## Summary

Criterion output includes confidence intervals, p-values, and regression detection. Understand these statistical measures to avoid chasing noise. Divan is a lighter alternative with attribute-based benchmarks. Both frameworks prevent the mistakes that make naive timing measurements unreliable.

## Code Examples

**Divan benchmarking framework with parameterized and simple benchmarks**

```rust
// Divan: attribute-based benchmarking (alternative to Criterion)
// Cargo.toml: divan = "0.1"

fn main() {
    divan::main();
}

#[divan::bench(args = [100, 1000, 10000])]
fn sort_vec(n: usize) -> Vec<i32> {
    let mut v: Vec<i32> = (0..n as i32).rev().collect();
    v.sort_unstable();
    v
}

#[divan::bench]
fn fibonacci() -> u64 {
    fn fib(n: u64) -> u64 {
        match n {
            0 | 1 => 1,
            n => fib(n - 1) + fib(n - 2),
        }
    }
    fib(divan::black_box(20))
}
```


## Resources

- [Criterion.rs — Comparing Functions](https://bheisler.github.io/criterion.rs/book/user_guide/comparing_functions.html) — Guide to comparing multiple implementations with Criterion
- [Divan — Rust Benchmarking Library](https://github.com/nvzqz/divan) — Attribute-based benchmarking framework for Rust

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*