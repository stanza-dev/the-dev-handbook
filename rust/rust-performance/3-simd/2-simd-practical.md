---
source_course: "rust-performance"
source_lesson: "rust-perf-simd-practical"
---

# SIMD Best Practices

## Data Alignment

```rust
// Aligned data is faster
#[repr(align(32))]  // AVX alignment
struct AlignedData {
    data: [f32; 8],
}

// Or use aligned_vec crate
use aligned_vec::AVec;
let data: AVec<f32, 32> = AVec::from_slice(32, &[1.0; 1000]);
```

## Helping Auto-Vectorization

```rust
// ✓ Good: Simple bounds, no dependencies
fn good(a: &mut [f32], b: &[f32]) {
    assert_eq!(a.len(), b.len());  // Helps optimizer
    for i in 0..a.len() {
        a[i] = b[i] * 2.0;
    }
}

// ✗ Bad: Dependencies between iterations
fn bad(a: &mut [f32]) {
    for i in 1..a.len() {
        a[i] = a[i - 1] * 2.0;  // Depends on previous
    }
}

// ✓ Better: Use iterators
fn best(data: &mut [f32]) {
    data.iter_mut().for_each(|x| *x *= 2.0);
}
```

## The `wide` Crate (Stable)

```rust
use wide::*;

// f32x8 on AVX, f32x4 on SSE
let a = f32x4::new([1.0, 2.0, 3.0, 4.0]);
let b = f32x4::new([5.0, 6.0, 7.0, 8.0]);

let sum = a + b;          // [6.0, 8.0, 10.0, 12.0]
let product = a * b;      // [5.0, 12.0, 21.0, 32.0]
let sqrt = a.sqrt();      // [1.0, 1.41.., 1.73.., 2.0]
let max = a.max(b);       // [5.0, 6.0, 7.0, 8.0]
```

## Use Cases for SIMD

- Image/video processing
- Audio processing
- Game physics
- Scientific computing
- Cryptography
- Compression

## Code Examples

**SIMD image brightening**

```rust
// Image processing example: Brighten pixels
fn brighten_scalar(pixels: &mut [u8], amount: u8) {
    for p in pixels {
        *p = p.saturating_add(amount);
    }
}

// SIMD version using wide crate
use wide::u8x16;

fn brighten_simd(pixels: &mut [u8], amount: u8) {
    let add = u8x16::splat(amount);
    
    let chunks = pixels.len() / 16;
    for i in 0..chunks {
        let offset = i * 16;
        let chunk = &pixels[offset..offset + 16];
        
        let v = u8x16::new([
            chunk[0], chunk[1], chunk[2], chunk[3],
            chunk[4], chunk[5], chunk[6], chunk[7],
            chunk[8], chunk[9], chunk[10], chunk[11],
            chunk[12], chunk[13], chunk[14], chunk[15],
        ]);
        
        let result = v.saturating_add(add);
        let arr = result.to_array();
        pixels[offset..offset + 16].copy_from_slice(&arr);
    }
    
    // Handle remainder
    for p in pixels[(chunks * 16)..].iter_mut() {
        *p = p.saturating_add(amount);
    }
}
```


---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*