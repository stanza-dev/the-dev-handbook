---
source_course: "rust-performance"
source_lesson: "rust-perf-data-locality"
---

# CPU Cache Basics

Memory access speed hierarchy:

| Level | Access Time | Size |
|-------|-------------|------|
| L1 Cache | ~1 ns | 32-64 KB |
| L2 Cache | ~4 ns | 256-512 KB |
| L3 Cache | ~10 ns | 4-50 MB |
| RAM | ~100 ns | GBs |

## Cache-Friendly Code

### Sequential Access is King

```rust
// Fast: Sequential access (cache-friendly)
let sum: i32 = vec.iter().sum();

// Slow: Random access (cache misses)
let mut sum = 0;
for i in random_indices {
    sum += vec[i];  // Cache miss likely
}
```

## Array of Structs vs Struct of Arrays

### Array of Structs (AoS)

```rust
// Traditional approach
struct Particle {
    x: f32, y: f32, z: f32,  // Position
    vx: f32, vy: f32, vz: f32,  // Velocity
}

let particles: Vec<Particle> = ...;

// If you only need positions, you still load velocities
for p in &particles {
    process_position(p.x, p.y, p.z);
    // Velocity data in cache but unused!
}
```

### Struct of Arrays (SoA)

```rust
// Cache-optimized for specific access patterns
struct Particles {
    xs: Vec<f32>,
    ys: Vec<f32>,
    zs: Vec<f32>,
    vxs: Vec<f32>,
    vys: Vec<f32>,
    vzs: Vec<f32>,
}

// Only loads what we need
for i in 0..particles.xs.len() {
    process_position(particles.xs[i], particles.ys[i], particles.zs[i]);
    // No velocity data polluting cache
}
```

## When to Use Each

| Pattern | AoS | SoA |
|---------|-----|-----|
| Access all fields together | ✓ | |
| Access specific fields | | ✓ |
| Iteration over all | | ✓ |
| Random access | ✓ | |
| Simple code | ✓ | |
| SIMD optimization | | ✓ |

## Code Examples

**Row vs column major access**

```rust
// Demonstrating cache effects
fn sum_rows(matrix: &[Vec<i32>]) -> i32 {
    // Fast: Row-major iteration matches memory layout
    let mut sum = 0;
    for row in matrix {
        for &val in row {
            sum += val;
        }
    }
    sum
}

fn sum_cols(matrix: &[Vec<i32>]) -> i32 {
    // Slow: Column-major access causes cache misses
    let mut sum = 0;
    let cols = matrix[0].len();
    for col in 0..cols {
        for row in matrix {
            sum += row[col];  // Jump around in memory
        }
    }
    sum
}

// Benchmark: sum_rows can be 10x+ faster for large matrices!
```


---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*