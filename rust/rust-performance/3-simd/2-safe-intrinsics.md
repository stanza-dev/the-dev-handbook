---
source_course: "rust-performance"
source_lesson: "rust-perf-safe-intrinsics"
---

# Safe std::arch Intrinsics

## Introduction

Rust 1.87 introduced one of the most significant performance-related changes in the language's history: most architecture-specific SIMD intrinsics in `std::arch` can now be called from safe code when the required target features are enabled at compile time. This dramatically reduces the need for `unsafe` blocks in SIMD code.

## Key Concepts

### Before Rust 1.87

```rust
use std::arch::x86_64::*;

// Had to be unsafe even with target feature enabled
#[target_feature(enable = "avx2")]
unsafe fn dot_product(a: &[f32; 8], b: &[f32; 8]) -> f32 {
    unsafe {
        let va = _mm256_loadu_ps(a.as_ptr());
        let vb = _mm256_loadu_ps(b.as_ptr());
        let mul = _mm256_mul_ps(va, vb);
        // ... reduction
    }
}
```

### After Rust 1.87: Safe Intrinsics

When target features are enabled at compile time (via `-C target-feature` or `target-cpu`), the intrinsics are safe to call:

```rust
use std::arch::x86_64::*;

// Compile with: RUSTFLAGS="-C target-feature=+avx2"
// No unsafe needed!
fn dot_product_safe(a: &[f32; 8], b: &[f32; 8]) -> f32 {
    let va = _mm256_loadu_ps(a.as_ptr());
    let vb = _mm256_loadu_ps(b.as_ptr());
    let mul = _mm256_mul_ps(va, vb);
    // ... safe reduction code
    0.0 // placeholder
}
```

### AVX-512 Target Features

AVX-512 target features were stabilized in Rust 1.89, and AVX-512fp16 plus AArch64 NEON fp16 features were stabilized in Rust 1.94. These can also be used safely when enabled.

## Real World Context

Libraries like `simd-json`, image codecs, and cryptographic implementations can now remove vast amounts of `unsafe` from their SIMD codepaths. This makes the code easier to audit and less prone to soundness bugs.

## Deep Dive

### `portable_simd` / `std::simd` Status

The higher-level `std::simd` API (portable SIMD) is still nightly-only as of Rust 1.94, with no stabilization timeline. For stable Rust, your options are:

1. **Safe `std::arch` intrinsics** (Rust 1.87+) — low-level but safe with target features.
2. **The `wide` crate** — stable, cross-platform SIMD wrapper.
3. **Auto-vectorization** — let the compiler do the work.

### The `wide` Crate

```rust
use wide::f32x8;

fn sum_wide(data: &[f32]) -> f32 {
    let mut acc = f32x8::ZERO;
    for chunk in data.chunks_exact(8) {
        let v = f32x8::new([
            chunk[0], chunk[1], chunk[2], chunk[3],
            chunk[4], chunk[5], chunk[6], chunk[7],
        ]);
        acc += v;
    }
    acc.reduce_add()
}
```

### Runtime Feature Detection

For portable binaries that adapt to the host CPU:

```rust
if is_x86_feature_detected!("avx2") {
    // Use AVX2 path (still needs unsafe for runtime-detected)
    unsafe { fast_avx2_path(&data) }
} else {
    scalar_fallback(&data)
}
```

Note: Runtime-detected intrinsics still require `unsafe` because the compiler cannot guarantee the feature is present at compile time.

## Common Pitfalls

- Confusing compile-time target features (safe) with runtime detection (still unsafe).
- Assuming `std::simd` / `portable_simd` is available on stable — it is nightly-only.
- Enabling AVX-512 globally when deploying to machines without AVX-512 support.

## Best Practices

- Use `-C target-cpu=native` for local builds to enable all safe intrinsics.
- Use the `wide` crate for portable, stable SIMD abstractions.
- Use runtime feature detection with fallbacks for distributed binaries.
- Prefer auto-vectorization first; drop to intrinsics only when the compiler fails.

## Summary

Rust 1.87's safe `std::arch` intrinsics are a game-changer for SIMD code, eliminating most `unsafe` blocks when target features are enabled at compile time. Combined with the `wide` crate for stable portable SIMD and auto-vectorization, Rust offers a complete toolkit for data-parallel performance. `portable_simd` remains nightly-only.

## Code Examples

**Stable SIMD dot product using the wide crate**

```rust
// Using the `wide` crate for stable, portable SIMD
use wide::f32x4;

fn dot_product(a: &[f32], b: &[f32]) -> f32 {
    assert_eq!(a.len(), b.len());
    let mut sum = f32x4::ZERO;
    let chunks = a.len() / 4;

    for i in 0..chunks {
        let off = i * 4;
        let va = f32x4::new([a[off], a[off+1], a[off+2], a[off+3]]);
        let vb = f32x4::new([b[off], b[off+1], b[off+2], b[off+3]]);
        sum += va * vb;
    }

    let mut result = sum.reduce_add();
    // Handle remainder
    for i in (chunks * 4)..a.len() {
        result += a[i] * b[i];
    }
    result
}
```


## Resources

- [Rust 1.87 Release Notes — Safe Arch Intrinsics](https://blog.rust-lang.org/2025/05/15/Rust-1.87.0/) — Announcement of safe std::arch intrinsics
- [wide crate](https://docs.rs/wide/latest/wide/) — Stable cross-platform SIMD types for Rust

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*