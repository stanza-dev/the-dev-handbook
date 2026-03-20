---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-simd-target-features"
---

# SIMD & Target Features

## Introduction
Single Instruction, Multiple Data (SIMD) allows a single CPU instruction to operate on multiple values simultaneously. Rust provides access to SIMD through `std::arch` intrinsics and the `#[target_feature]` attribute. Understanding these is essential for writing high-performance code that takes advantage of modern CPU capabilities.

## Key Concepts
- **SIMD**: Processing multiple data elements with a single instruction (e.g., adding 4 floats at once).
- **`std::arch`**: Module providing architecture-specific intrinsics as safe wrappers around CPU instructions.
- **`#[target_feature(enable = "...")]`**: Attribute that enables specific CPU features for a function. The function becomes `unsafe` to call because calling it on a CPU without the feature is UB.
- **Runtime feature detection**: `is_x86_feature_detected!` checks at runtime whether the CPU supports a feature.

## Real World Context
Image processing, audio codecs, cryptography, game physics, and machine learning all use SIMD for 2-10x speedups. Rust's standard library uses SIMD internally for `str::contains`, `[u8]::contains`, and hash map probing.

## Deep Dive

### Using `std::arch` Intrinsics

The `std::arch` module provides typed wrappers for CPU instructions:

```rust
#[cfg(target_arch = "x86_64")]
use std::arch::x86_64::*;

#[target_feature(enable = "avx2")]
unsafe fn sum_avx2(data: &[f32]) -> f32 {
    let mut sum = _mm256_setzero_ps(); // 8 floats = 0.0

    for chunk in data.chunks_exact(8) {
        let v = _mm256_loadu_ps(chunk.as_ptr());
        sum = _mm256_add_ps(sum, v); // Add 8 floats at once
    }

    // Horizontal sum of the 8 lanes
    let mut result = [0.0f32; 8];
    _mm256_storeu_ps(result.as_mut_ptr(), sum);
    result.iter().sum()
}
```

The `#[target_feature(enable = "avx2")]` enables AVX2 instructions for this function. The function is unsafe because calling it on a CPU without AVX2 is undefined behavior.

### Runtime Feature Detection

Use runtime checks to safely dispatch to SIMD code:

```rust
fn fast_sum(data: &[f32]) -> f32 {
    #[cfg(target_arch = "x86_64")]
    {
        if is_x86_feature_detected!("avx2") {
            // SAFETY: We just checked that AVX2 is available
            return unsafe { sum_avx2(data) };
        }
    }
    // Fallback: scalar implementation
    data.iter().sum()
}
```

The `is_x86_feature_detected!` macro queries CPUID at runtime. The result is cached after the first call.

### Portable SIMD (Nightly)

The `std::simd` module (nightly) provides portable SIMD that works across architectures:

```rust
#![feature(portable_simd)]
use std::simd::prelude::*;

fn add_arrays(a: &[f32; 4], b: &[f32; 4]) -> [f32; 4] {
    let va = f32x4::from_array(*a);
    let vb = f32x4::from_array(*b);
    (va + vb).to_array()
}
```

Portable SIMD compiles to the best available instructions on any platform.

## Common Pitfalls
1. **Forgetting runtime feature detection** — Calling a `#[target_feature]` function on a CPU that lacks the feature is UB. Always check with `is_x86_feature_detected!`.
2. **Ignoring alignment** — Some SIMD load instructions require aligned pointers. Use `_mm256_loadu_ps` (unaligned) instead of `_mm256_load_ps` (requires 32-byte alignment) unless you've ensured alignment.
3. **Not providing a scalar fallback** — Your code must work on CPUs without SIMD. Always provide a non-SIMD fallback path.

## Best Practices
1. **Use `is_x86_feature_detected!` for dynamic dispatch** — Check once, cache the function pointer for repeated calls.
2. **Benchmark both SIMD and scalar** — For small data, the overhead of SIMD setup may exceed the benefit.
3. **Prefer portable SIMD when available** — It works across architectures without architecture-specific code.

## Summary
- SIMD processes multiple data elements with a single instruction.
- `std::arch` provides typed intrinsics; `#[target_feature]` enables them.
- Always use runtime feature detection before calling SIMD functions.
- Provide scalar fallbacks for CPUs without SIMD support.
- Portable SIMD (nightly) provides cross-architecture SIMD.

## Code Examples

**SIMD byte counting with SSE2 — processes 16 bytes per iteration with runtime feature detection and scalar fallback**

```rust
#[cfg(target_arch = "x86_64")]
use std::arch::x86_64::*;

/// Count bytes equal to `needle` in `haystack` using SIMD.
/// Falls back to scalar if SSE2 is not available.
fn count_byte(haystack: &[u8], needle: u8) -> usize {
    #[cfg(target_arch = "x86_64")]
    {
        if is_x86_feature_detected!("sse2") {
            // SAFETY: SSE2 is available
            return unsafe { count_byte_sse2(haystack, needle) };
        }
    }
    // Scalar fallback
    haystack.iter().filter(|&&b| b == needle).count()
}

#[cfg(target_arch = "x86_64")]
#[target_feature(enable = "sse2")]
unsafe fn count_byte_sse2(haystack: &[u8], needle: u8) -> usize {
    let needle_v = _mm_set1_epi8(needle as i8); // Broadcast needle to 16 lanes
    let mut count = 0usize;

    for chunk in haystack.chunks_exact(16) {
        let data = _mm_loadu_si128(chunk.as_ptr() as *const __m128i);
        let cmp = _mm_cmpeq_epi8(data, needle_v); // Compare 16 bytes at once
        let mask = _mm_movemask_epi8(cmp) as u32;
        count += mask.count_ones() as usize;
    }

    // Handle remaining bytes
    let remainder = haystack.len() % 16;
    if remainder > 0 {
        let tail = &haystack[haystack.len() - remainder..];
        count += tail.iter().filter(|&&b| b == needle).count();
    }

    count
}
```


## Resources

- [std::arch — Platform-specific intrinsics](https://doc.rust-lang.org/std/arch/index.html) — API reference for architecture-specific SIMD intrinsics

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*