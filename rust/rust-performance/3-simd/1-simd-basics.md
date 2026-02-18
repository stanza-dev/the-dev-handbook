---
source_course: "rust-performance"
source_lesson: "rust-perf-simd-basics"
---

# SIMD: Single Instruction, Multiple Data

Process multiple values with one CPU instruction.

```
// Scalar: 4 operations
a[0] + b[0] → c[0]
a[1] + b[1] → c[1]
a[2] + b[2] → c[2]
a[3] + b[3] → c[3]

// SIMD: 1 operation
[a0,a1,a2,a3] + [b0,b1,b2,b3] → [c0,c1,c2,c3]
```

## std::simd (Nightly)

```rust
#![feature(portable_simd)]
use std::simd::*;

fn sum_simd(data: &[f32]) -> f32 {
    let mut sum = f32x4::splat(0.0);
    
    for chunk in data.chunks_exact(4) {
        let v = f32x4::from_slice(chunk);
        sum += v;
    }
    
    sum.reduce_sum()
}
```

## Auto-Vectorization

The compiler can vectorize simple loops:

```rust
// Likely to be auto-vectorized
fn add_arrays(a: &[f32], b: &[f32], out: &mut [f32]) {
    for i in 0..out.len() {
        out[i] = a[i] + b[i];
    }
}
```

## Checking Vectorization

```bash
# See assembly
RUSTFLAGS="--emit=asm -C target-cpu=native" cargo build --release

# Look for vector instructions:
# SSE: xmm registers, movaps, addps
# AVX: ymm registers, vaddps
# AVX-512: zmm registers
```

## Enabling SIMD

```toml
# .cargo/config.toml
[build]
rustflags = ["-C", "target-cpu=native"]

# Or in Cargo.toml
[profile.release]
lto = true  # Helps with inlining for vectorization
```

## Code Examples

**SIMD dot product**

```rust
// Using the `wide` crate (stable Rust)
use wide::*;

fn dot_product_simd(a: &[f32], b: &[f32]) -> f32 {
    assert_eq!(a.len(), b.len());
    
    let mut sum = f32x4::ZERO;
    let chunks = a.len() / 4;
    
    for i in 0..chunks {
        let offset = i * 4;
        let va = f32x4::new([
            a[offset], a[offset + 1], 
            a[offset + 2], a[offset + 3]
        ]);
        let vb = f32x4::new([
            b[offset], b[offset + 1], 
            b[offset + 2], b[offset + 3]
        ]);
        sum += va * vb;
    }
    
    // Handle remainder
    let mut result: f32 = sum.reduce_add();
    for i in (chunks * 4)..a.len() {
        result += a[i] * b[i];
    }
    
    result
}
```


---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*