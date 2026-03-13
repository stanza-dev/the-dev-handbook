---
source_course: "rust-performance"
source_lesson: "rust-perf-simd-basics"
---

# SIMD Fundamentals

## Introduction

SIMD (Single Instruction, Multiple Data) processes multiple values with one CPU instruction. Instead of adding four pairs of numbers in four operations, SIMD adds all four pairs in one operation — a potential 4x speedup.

## Key Concepts

### How SIMD Works

```
Scalar: 4 operations
a[0] + b[0] -> c[0]
a[1] + b[1] -> c[1]
a[2] + b[2] -> c[2]
a[3] + b[3] -> c[3]

SIMD: 1 operation
[a0,a1,a2,a3] + [b0,b1,b2,b3] -> [c0,c1,c2,c3]
```

### SIMD Instruction Sets

| Instruction Set | Register Width | f32 per op |
|----------------|---------------|------------|
| SSE/SSE2 | 128-bit (xmm) | 4 |
| AVX/AVX2 | 256-bit (ymm) | 8 |
| AVX-512 | 512-bit (zmm) | 16 |
| NEON (ARM) | 128-bit | 4 |

### Auto-Vectorization

The compiler can automatically vectorize simple loops:

```rust
// Likely to auto-vectorize
fn add_arrays(a: &[f32], b: &[f32], out: &mut [f32]) {
    assert_eq!(a.len(), b.len());
    assert_eq!(a.len(), out.len());
    for i in 0..out.len() {
        out[i] = a[i] + b[i];
    }
}
```

## Real World Context

SIMD is critical in image processing, audio codecs, scientific computing, game physics, cryptography, and compression. Libraries like `simd-json` parse JSON 2-4x faster using SIMD.

## Deep Dive

### Checking Vectorization

```bash
# Check if the compiler vectorized your loop
RUSTFLAGS="--emit=asm -C target-cpu=native" cargo build --release

# Look for vector instructions:
# SSE: movaps, addps, mulps (xmm registers)
# AVX: vaddps, vmulps (ymm registers)
# AVX-512: vaddps (zmm registers)
```

Or use Godbolt (compiler explorer) to inspect assembly online.

### Enabling SIMD

```toml
# .cargo/config.toml
[build]
rustflags = ["-C", "target-cpu=native"]
```

This enables all SIMD instructions your CPU supports. For portable binaries, specify a baseline like `target-cpu=x86-64-v3` (AVX2).

## Common Pitfalls

- Assuming the compiler auto-vectorizes everything — check the assembly.
- Using `target-cpu=native` in distributed binaries — it will crash on older CPUs.
- Loop-carried dependencies prevent vectorization (each iteration depending on the previous).

## Best Practices

- Start with auto-vectorization: write simple loops and check the assembly.
- Use `target-cpu=native` for benchmarks and local builds.
- Use `target-cpu=x86-64-v3` as a reasonable portable AVX2 baseline.
- When auto-vectorization fails, use explicit SIMD intrinsics.

## Summary

SIMD processes multiple data elements per instruction, delivering significant speedups for data-parallel workloads. The compiler can auto-vectorize simple loops, but you must enable the right target features and verify the output assembly.

## Code Examples

**Patterns that help and hinder auto-vectorization**

```rust
// Helping the compiler auto-vectorize

// Good: simple loop, no dependencies between iterations
fn scale(data: &mut [f32], factor: f32) {
    for x in data.iter_mut() {
        *x *= factor;
    }
}

// Bad: loop-carried dependency prevents vectorization
fn prefix_sum(data: &mut [f32]) {
    for i in 1..data.len() {
        data[i] += data[i - 1]; // depends on previous iteration
    }
}

// Good: assert helps optimizer prove no aliasing
fn add_slices(a: &[f32], b: &[f32], out: &mut [f32]) {
    assert_eq!(a.len(), b.len());
    assert_eq!(a.len(), out.len());
    for i in 0..a.len() {
        out[i] = a[i] + b[i];
    }
}
```


## Resources

- [Build Configuration — CPU Specific Instructions](https://nnethercote.github.io/perf-book/build-configuration.html) — Rust Performance Book section on target-cpu and SIMD instruction sets
- [Compiler Explorer (Godbolt)](https://godbolt.org/) — Inspect compiler output to verify vectorization

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*