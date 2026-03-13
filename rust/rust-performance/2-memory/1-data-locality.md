---
source_course: "rust-performance"
source_lesson: "rust-perf-data-locality"
---

# Data Locality & CPU Cache

## Introduction

The single biggest factor in data-processing performance is not the algorithm — it is how well your data fits the CPU cache. A cache miss to RAM costs ~100x more than a cache hit.

## Key Concepts

### Cache Hierarchy

| Level | Latency | Typical Size |
|-------|---------|-------------|
| L1 | ~1 ns | 32-64 KB |
| L2 | ~4 ns | 256-512 KB |
| L3 | ~10 ns | 4-50 MB |
| RAM | ~100 ns | GBs |

The CPU loads data in **cache lines** (typically 64 bytes). When you access one byte, the surrounding 64 bytes come along for free.

### Sequential vs Random Access

```rust
// Fast: sequential (cache-friendly)
let sum: i32 = vec.iter().sum();

// Slow: random access (cache misses)
for i in random_indices {
    sum += vec[i]; // likely cache miss
}
```

## Real World Context

Game engines use SoA layouts for particle systems. Database engines organize columns contiguously. Any system processing millions of items must think about data locality.

## Deep Dive

### Array of Structs (AoS) vs Struct of Arrays (SoA)

**AoS (traditional):**

```rust
struct Particle {
    x: f32, y: f32, z: f32,
    vx: f32, vy: f32, vz: f32,
}
let particles: Vec<Particle> = ...;

// Reading only positions loads velocity data too
for p in &particles {
    process_position(p.x, p.y, p.z);
}
```

**SoA (cache-optimized):**

```rust
struct Particles {
    xs: Vec<f32>, ys: Vec<f32>, zs: Vec<f32>,
    vxs: Vec<f32>, vys: Vec<f32>, vzs: Vec<f32>,
}

// Only loads position data into cache
for i in 0..particles.xs.len() {
    process_position(particles.xs[i], particles.ys[i], particles.zs[i]);
}
```

### When to Use Each

| Pattern | AoS | SoA |
|---------|-----|-----|
| Access all fields together | Best | |
| Access specific fields in hot loop | | Best |
| SIMD optimization | | Best |
| Simple code | Best | |

## Common Pitfalls

- Using `LinkedList` — it scatters nodes across the heap. `Vec` is almost always faster.
- Iterating a `HashMap` for performance-critical paths — entries are not contiguous.
- Column-major iteration of row-major data (or vice versa) — can be 10x slower for large matrices.

## Best Practices

- Default to `Vec` for collections. It has the best cache behavior.
- Use SoA when hot loops access only a subset of fields.
- Prefer iterators over indexing — they help the compiler vectorize and prefetch.
- Benchmark both AoS and SoA; the difference depends on access patterns.

## Summary

Cache-friendly data layout is the foundation of high-performance Rust. Sequential access over contiguous memory (Vec, SoA) wins over pointer-chasing (LinkedList, HashMap) by orders of magnitude. Always measure with real workloads.

## Code Examples

**Row-major vs column-major access showing cache effects**

```rust
// Demonstrating cache effects: row-major vs column-major
fn sum_rows(matrix: &[Vec<i32>]) -> i32 {
    // Fast: matches memory layout
    let mut sum = 0;
    for row in matrix {
        for &val in row {
            sum += val;
        }
    }
    sum
}

fn sum_cols(matrix: &[Vec<i32>]) -> i32 {
    // Slow: jumps around in memory
    let mut sum = 0;
    let cols = matrix[0].len();
    for col in 0..cols {
        for row in matrix {
            sum += row[col]; // cache miss likely
        }
    }
    sum
}
// sum_rows can be 10x+ faster for large matrices!
```


## Resources

- [What Every Programmer Should Know About Memory](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf) — Classic paper on memory hierarchy and cache optimization
- [The Rust Performance Book — Type Sizes](https://nnethercote.github.io/perf-book/type-sizes.html) — Practical advice on data layout in Rust

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*