---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-intrinsics"
---

# Compiler Intrinsics & Optimization Hints

## Introduction
Compiler intrinsics are special functions that the compiler translates directly into optimized machine code. They provide access to CPU features and optimization hints that normal Rust code cannot express. While many intrinsics are nightly-only, Rust provides stable alternatives for the most common use cases.

## Key Concepts
- **Intrinsic**: A function known to the compiler that generates specialized CPU instructions rather than a normal function call.
- **Branch hint**: A directive telling the compiler which branch of an `if` statement is more likely, so it can optimize the common path.
- **`#[cold]`**: An attribute marking a function as rarely called, so the compiler optimizes callers to avoid branching to it.
- **`std::hint`**: A module providing stable optimization hints like `spin_loop`, `black_box`, and `unreachable_unchecked`.

## Real World Context
High-performance libraries like `hashbrown` (Rust's HashMap implementation) use intrinsics for SIMD operations and branch hints. Network servers use `#[cold]` on error paths to keep the hot path fast. Benchmarking frameworks use `black_box` to prevent dead code elimination.

## Deep Dive

### Branch Hints (Nightly)

The `likely` and `unlikely` intrinsics tell the compiler which branch is expected to be taken:

```rust
#![feature(core_intrinsics)]
use std::intrinsics::{likely, unlikely};

fn process(x: i32) {
    if unsafe { unlikely(x == 0) } {
        handle_error(); // Rarely taken
    } else {
        process_value(x); // Hot path
    }
}
```

The compiler uses these hints to arrange code so the likely path has fewer branches and better cache locality.

### Stable Alternative: `#[cold]`

On stable Rust, use `#[cold]` to achieve a similar effect:

```rust
#[cold]
#[inline(never)]
fn handle_error(msg: &str) -> ! {
    eprintln!("Error: {msg}");
    std::process::exit(1);
}

fn process(x: i32) {
    if x == 0 {
        handle_error("unexpected zero"); // Compiler optimizes this as unlikely
    }
    // Hot path continues here
}
```

When a function called from a branch is `#[cold]`, the compiler treats that branch as unlikely.

### `hint::select_unpredictable` (Stable since rust 1.88)

For branchless selection where the branch is truly unpredictable:

```rust
use std::hint::select_unpredictable;

fn min_branchless(a: i32, b: i32) -> i32 {
    // Tells the compiler: don't try to predict this branch
    select_unpredictable(a < b, a, b)
}
```

This is useful for cases where branch prediction consistently fails (random data, hash tables). The compiler may use conditional moves (cmov) instead of branches.

### Stable Hints in `std::hint`

The `std::hint` module provides several useful stable functions:

```rust
use std::hint;

// 1. spin_loop: CPU hint for busy-wait loops
fn wait_for_flag(flag: &std::sync::atomic::AtomicBool) {
    while !flag.load(std::sync::atomic::Ordering::Acquire) {
        hint::spin_loop(); // Reduces power, yields to SMT sibling
    }
}

// 2. black_box: Prevent compiler from optimizing away a value
fn benchmark() {
    let result = expensive_computation();
    hint::black_box(result); // Compiler cannot eliminate this
}

// 3. unreachable_unchecked: Assert impossible branch
fn get_value(opt: Option<i32>) -> i32 {
    match opt {
        Some(v) => v,
        None => unsafe { hint::unreachable_unchecked() },
    }
}
```

`unreachable_unchecked` is UB if reached — only use it when you can prove the branch is impossible.

### Type Intrinsics (Stable)

```rust
use std::mem::{size_of, align_of, needs_drop};

println!("Size of String: {}", size_of::<String>());   // 24
println!("Alignment: {}", align_of::<String>());        // 8
println!("Needs drop: {}", needs_drop::<String>());     // true
println!("Needs drop: {}", needs_drop::<i32>());        // false
```

These are resolved at compile time and have zero runtime cost.

## Common Pitfalls
1. **Using `unreachable_unchecked` when the branch is reachable** — This is instant UB. Prefer `unreachable!()` (which panics) unless you can formally prove the code is unreachable.
2. **Overusing branch hints** — Modern CPUs have good branch predictors. Only add hints in hot loops where profiling shows branch misprediction.
3. **Forgetting `black_box` in benchmarks** — Without it, the compiler may optimize away the code you're trying to measure.

## Best Practices
1. **Profile before optimizing** — Use `perf stat` or `cargo bench` to identify branch misprediction hotspots before adding hints.
2. **Use `#[cold]` on error paths** — This is the simplest and most effective stable optimization for error handling.
3. **Use `select_unpredictable` for data-dependent branches** — Hash map lookups and binary searches with random data benefit from branchless selection.

## Summary
- Intrinsics generate specialized CPU instructions for optimization.
- `likely`/`unlikely` are nightly-only; use `#[cold]` on stable as an alternative.
- `hint::select_unpredictable` (stable since 1.88) enables branchless selection.
- `black_box` prevents dead code elimination in benchmarks.
- `unreachable_unchecked` is UB if reached — only use with formal proof of unreachability.

## Code Examples

**Stable optimization hints in Rust — #[cold] for error paths, #[inline(always)] for hot code, black_box for benchmarks, and spin_loop for busy waiting**

```rust
// Stable optimization hints

// 1. #[cold] for error paths
#[cold]
#[inline(never)]
fn handle_allocation_failure() -> ! {
    eprintln!("Out of memory");
    std::process::exit(1);
}

// 2. #[inline(always)] for hot functions
#[inline(always)]
fn fast_hash(key: u64) -> u64 {
    key.wrapping_mul(0x517cc1b727220a95)
}

// 3. black_box for benchmarks
use std::hint::black_box;

fn benchmark_sort() {
    let mut data = black_box(vec![5, 3, 1, 4, 2]);
    data.sort();
    black_box(data); // Prevent dead code elimination
}

// 4. spin_loop for busy waiting
use std::sync::atomic::{AtomicBool, Ordering};

fn spin_wait(ready: &AtomicBool) {
    while !ready.load(Ordering::Acquire) {
        std::hint::spin_loop();
    }
}
```


## Resources

- [std::hint module](https://doc.rust-lang.org/std/hint/index.html) — Complete API reference for Rust's optimization hint functions
- [std::intrinsics module (nightly)](https://doc.rust-lang.org/std/intrinsics/index.html) — Nightly compiler intrinsics documentation

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*