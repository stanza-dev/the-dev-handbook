---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-intrinsics"
---

# Compiler Intrinsics

Intrinsics are functions known to the compiler that generate specialized code.

## Branch Hints (Nightly)

```rust
#![feature(core_intrinsics)]
use std::intrinsics::{likely, unlikely};

fn process(x: i32) {
    if unsafe { unlikely(x == 0) } {
        handle_error();  // Rarely taken
    } else {
        // Hot path - compiler optimizes for this
        process_value(x);
    }
}
```

## Stable Alternatives

```rust
// Use cold attribute for unlikely functions
#[cold]
fn handle_error() {
    panic!("Error!");
}

// Use inline for hot functions
#[inline(always)]
fn hot_path(x: i32) -> i32 {
    x * 2
}
```

## Prefetching

```rust
#![feature(core_intrinsics)]
use std::intrinsics::prefetch_read_data;

fn process_array(data: &[i32]) {
    for i in 0..data.len() {
        // Prefetch future data
        if i + 16 < data.len() {
            unsafe {
                prefetch_read_data(&data[i + 16], 3);
            }
        }
        process(data[i]);
    }
}
```

## Atomic Intrinsics

Most atomic operations are available through `std::sync::atomic`, but intrinsics provide more:

```rust
#![feature(core_intrinsics)]
use std::intrinsics::atomic_fence_acqrel;

unsafe {
    atomic_fence_acqrel();  // Memory fence
}
```

## Type Intrinsics

```rust
use std::mem::{size_of, align_of, needs_drop};

println!("Size: {}", size_of::<String>());
println!("Align: {}", align_of::<String>());
println!("Needs drop: {}", needs_drop::<String>());
```

## Code Examples

**Stable optimization hints**

```rust
// Stable hint alternatives

// 1. Using cold for error paths
#[cold]
#[inline(never)]
fn error_handler(msg: &str) -> ! {
    eprintln!("Error: {msg}");
    std::process::exit(1);
}

// 2. Using inline for hot paths
#[inline(always)]
fn fast_multiply(a: i32, b: i32) -> i32 {
    a * b
}

// 3. Unreachable hint for impossible branches
fn get_value(opt: Option<i32>) -> i32 {
    match opt {
        Some(v) => v,
        None => unsafe { std::hint::unreachable_unchecked() },
    }
}

// 4. Black box to prevent optimization
use std::hint::black_box;

fn benchmark() {
    let result = expensive_computation();
    black_box(result);  // Prevents dead code elimination
}

// 5. Spin loop hint
fn busy_wait(flag: &std::sync::atomic::AtomicBool) {
    while !flag.load(std::sync::atomic::Ordering::Acquire) {
        std::hint::spin_loop();
    }
}
```


---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*