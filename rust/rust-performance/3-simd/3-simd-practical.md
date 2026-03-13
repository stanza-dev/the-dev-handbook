---
source_course: "rust-performance"
source_lesson: "rust-perf-simd-practical"
---

# Practical SIMD Patterns

## Introduction

Knowing what SIMD is differs from knowing when and how to apply it effectively. This lesson covers practical patterns for data alignment, helping auto-vectorization, and real use cases.

## Key Concepts

### Data Alignment

SIMD instructions perform best on aligned data:

```rust
#[repr(align(32))] // AVX alignment
struct AlignedBuffer {
    data: [f32; 8],
}
```

Misaligned loads work but may be slower, especially on older CPUs.

### Helping Auto-Vectorization

The compiler vectorizes better when it can prove safety:

```rust
// Good: assert proves lengths match
fn add(a: &[f32], b: &[f32], out: &mut [f32]) {
    assert_eq!(a.len(), b.len());
    assert_eq!(a.len(), out.len());
    for i in 0..a.len() {
        out[i] = a[i] + b[i];
    }
}

// Good: iterators help prove no aliasing
fn scale(data: &mut [f32], factor: f32) {
    data.iter_mut().for_each(|x| *x *= factor);
}

// Bad: loop-carried dependency blocks vectorization
fn running_sum(data: &mut [f32]) {
    for i in 1..data.len() {
        data[i] += data[i - 1]; // can't parallelize
    }
}
```

## Real World Context

Image processing is the classic SIMD use case. Brightening every pixel, applying convolution filters, or converting color spaces are all embarrassingly parallel over pixel data.

## Deep Dive

### Use Cases for Explicit SIMD

- **Image/Video processing**: per-pixel operations
- **Audio processing**: sample-level DSP
- **Scientific computing**: matrix operations, FFT
- **Cryptography**: AES-NI, SHA extensions
- **Compression**: pattern matching in zstd, snappy
- **JSON parsing**: simd-json processes 2-4x faster

### `chunks_exact` for SIMD-Friendly Iteration

```rust
fn process_simd(data: &mut [f32], factor: f32) {
    // Process 4 elements at a time
    let (chunks, remainder) = data.split_at_mut(
        data.len() - data.len() % 4
    );

    // SIMD path: compiler can vectorize this
    for chunk in chunks.chunks_exact_mut(4) {
        chunk[0] *= factor;
        chunk[1] *= factor;
        chunk[2] *= factor;
        chunk[3] *= factor;
    }

    // Scalar remainder
    for x in remainder {
        *x *= factor;
    }
}
```

### Avoiding Branches in SIMD

Branches prevent vectorization. Use arithmetic instead:

```rust
// Bad: branch in hot loop
for x in data.iter_mut() {
    if *x < 0.0 { *x = 0.0; } // branch prevents SIMD
}

// Good: branchless equivalent
for x in data.iter_mut() {
    *x = x.max(0.0); // compiles to maxps SIMD instruction
}
```

## Common Pitfalls

- Adding branches inside vectorizable loops — use `min`, `max`, `clamp` instead.
- Mixing data types in a loop — SIMD operates on uniform types.
- Not checking assembly to verify vectorization actually happened.

## Best Practices

- Use `chunks_exact` / `chunks_exact_mut` for natural SIMD-width processing.
- Replace branches with arithmetic equivalents (max, min, clamp, select).
- Use iterators — they encode the no-aliasing guarantee that enables vectorization.
- Profile both scalar and SIMD versions; auto-vectorization sometimes wins over manual SIMD.

## Summary

Practical SIMD optimization focuses on data alignment, helping auto-vectorization through assertions and iterators, eliminating branches in hot loops, and choosing the right abstraction level (auto-vectorize, `wide` crate, or raw intrinsics).

## Code Examples

**Branchless SIMD-friendly patterns for image and numeric processing**

```rust
// Branchless SIMD-friendly clamp
fn clamp_values(data: &mut [f32], min: f32, max: f32) {
    // This compiles to SIMD min/max instructions
    for x in data.iter_mut() {
        *x = x.clamp(min, max);
    }
}

// Image brightening: classic SIMD use case
fn brighten(pixels: &mut [u8], amount: u8) {
    for p in pixels.iter_mut() {
        *p = p.saturating_add(amount); // SIMD saturating add
    }
}
```


## Resources

- [simd-json](https://github.com/simd-lite/simd-json) — SIMD-accelerated JSON parser for Rust
- [Build Configuration — CPU Specific Instructions](https://nnethercote.github.io/perf-book/build-configuration.html) — Rust Performance Book on enabling SIMD via target-cpu flags

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*