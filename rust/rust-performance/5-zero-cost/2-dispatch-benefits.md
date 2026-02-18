---
source_course: "rust-performance"
source_lesson: "rust-perf-static-dispatch-benefits"
---

# Iterator Chains Are Zero-Cost

Rust iterators compile down to optimal code:

```rust
// High-level, readable
let sum: i32 = (0..1000)
    .filter(|x| x % 2 == 0)
    .map(|x| x * x)
    .sum();

// Compiles to roughly equivalent of:
let mut sum = 0;
for i in 0..1000 {
    if i % 2 == 0 {
        sum += i * i;
    }
}
```

## Why It's Zero-Cost

1. **Static Dispatch**: Each iterator adapter is a concrete type
2. **Inlining**: Small closures are inlined
3. **Fusion**: Multiple operations combined into one loop
4. **No Allocation**: Lazy evaluation, no intermediate collections

## Proving Zero Cost

```rust
// Check the assembly
pub fn manual_sum() -> i32 {
    let mut sum = 0;
    for i in 0..1000 {
        if i % 2 == 0 {
            sum += i * i;
        }
    }
    sum
}

pub fn iterator_sum() -> i32 {
    (0..1000)
        .filter(|x| x % 2 == 0)
        .map(|x| x * x)
        .sum()
}

// With --release, these generate nearly identical assembly!
```

## When Abstraction Has Cost

```rust
// This has overhead:
let boxed: Box<dyn Iterator<Item = i32>> = Box::new(0..1000);
// Dynamic dispatch, allocation

// This is zero-cost:
let iter = 0..1000;  // Concrete type: Range<i32>
```

## The Rust Philosophy

"What you don't use, you don't pay for. And what you do use, you couldn't hand code any better." - Bjarne Stroustrup (C++, but applies to Rust too)

## Code Examples

**Lazy iterator efficiency**

```rust
// Complex iterator chain - still zero cost!
fn process_data(data: &[i32]) -> Vec<i32> {
    data.iter()
        .copied()                    // &i32 -> i32
        .filter(|&x| x > 0)          // Keep positives
        .map(|x| x * 2)              // Double
        .take(100)                   // First 100
        .filter(|&x| x % 3 == 0)     // Divisible by 3
        .collect()                   // Allocates once!
}

// Only ONE allocation (the final Vec)
// Only ONE pass through data (up to 100 matching items)
// All filtering happens during iteration

// Compare to naive approach:
fn process_data_naive(data: &[i32]) -> Vec<i32> {
    let positives: Vec<_> = data.iter().filter(|&&x| x > 0).collect();  // Alloc 1
    let doubled: Vec<_> = positives.iter().map(|&x| x * 2).collect();   // Alloc 2
    let first_100: Vec<_> = doubled.into_iter().take(100).collect();    // Alloc 3
    first_100.into_iter().filter(|&x| x % 3 == 0).collect()             // Alloc 4
}
```


---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*