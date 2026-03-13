---
source_course: "rust-performance"
source_lesson: "rust-perf-iterators-dispatch"
---

# Iterators & Static Dispatch

## Introduction

Rust's iterator chains compile to the same machine code as hand-written loops. This is the quintessential zero-cost abstraction — high-level expressiveness with low-level performance.

## Key Concepts

### Iterators Are Zero-Cost

```rust
// High-level, expressive
let sum: i32 = data.iter()
    .filter(|&&x| x > 0)
    .map(|&x| x * 2)
    .sum();

// Compiles to roughly the same code as:
let mut sum = 0;
for &x in &data {
    if x > 0 {
        sum += x * 2;
    }
}
```

### Monomorphization

Generics in Rust use **monomorphization**: the compiler generates specialized code for each concrete type.

```rust
fn add<T: std::ops::Add<Output = T>>(a: T, b: T) -> T {
    a + b
}

// Compiler generates:
// fn add_i32(a: i32, b: i32) -> i32 { a + b }
// fn add_f64(a: f64, b: f64) -> f64 { a + b }
```

No runtime dispatch, no vtable lookup — just direct function calls.

## Real World Context

Iterator chains are the idiomatic way to process collections in Rust. Libraries like `rayon` extend iterators with parallelism (`.par_iter()`) while maintaining the zero-cost principle.

## Deep Dive

### Static vs Dynamic Dispatch

```rust
// Static dispatch: monomorphized, inlined (zero cost)
fn process(iter: impl Iterator<Item = i32>) -> i32 {
    iter.sum()
}

// Dynamic dispatch: vtable lookup per call (has cost)
fn process_dyn(iter: &mut dyn Iterator<Item = i32>) -> i32 {
    iter.sum()
}
```

### When Abstraction Has Cost

Monomorphization can increase binary size:

```rust
fn process<T: Display>(items: &[T]) { ... }

// If called with 10 different types, 10 copies of process exist
// in the binary. This is the monomorphization bloat tradeoff.
```

### collect() and Allocation

Iterator chains are lazy until consumed. But `.collect()` allocates:

```rust
// Allocates a new Vec (unavoidable if you need one)
let doubled: Vec<i32> = data.iter().map(|&x| x * 2).collect();

// Better if you just need to iterate:
for x in data.iter().map(|&x| x * 2) {
    process(x); // No allocation
}
```

### Benchmark Proof

Criterion benchmarks consistently show that iterator chains match or beat manual loops because the compiler can fuse, vectorize, and optimize the entire chain as a unit.

## Common Pitfalls

- Calling `.collect()` just to iterate the result — use the iterator directly.
- Using `dyn Iterator` when `impl Iterator` works — dynamic dispatch adds overhead.
- Assuming monomorphization is always free — binary size grows with each specialization.

## Best Practices

- Use iterator chains for data processing — they are idiomatic and zero-cost.
- Prefer `impl Trait` over `dyn Trait` when the type is known at compile time.
- Use `for_each` instead of a `for` loop when the closure is simple (compiler hint).
- Watch binary size in generic-heavy code; consider `dyn Trait` for cold paths.

## Summary

Rust's iterators are zero-cost abstractions: they compile to the same code as hand-written loops. Monomorphization eliminates runtime dispatch for generics. Use `impl Trait` for static dispatch and `dyn Trait` only when you need runtime polymorphism or want to reduce binary size.

## Code Examples

**Iterator chain and manual loop produce identical machine code**

```rust
// Iterator chain vs manual loop — identical performance
fn sum_even_doubled_iter(data: &[i32]) -> i32 {
    data.iter()
        .filter(|&&x| x % 2 == 0)
        .map(|&x| x * 2)
        .sum()
}

fn sum_even_doubled_loop(data: &[i32]) -> i32 {
    let mut sum = 0;
    for &x in data {
        if x % 2 == 0 {
            sum += x * 2;
        }
    }
    sum
}

// Both compile to the same optimized machine code.
// The iterator version is more idiomatic and equally fast.
```


## Resources

- [Rust Performance — Iterators](https://nnethercote.github.io/perf-book/iterators.html) — Performance analysis of Rust iterators
- [Abstraction without overhead: traits in Rust](https://blog.rust-lang.org/2015/05/11/traits.html) — Rust blog post on zero-cost trait abstractions

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*