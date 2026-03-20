---
source_course: "rust-embedded"
source_lesson: "rust-embedded-alloc-crate"
---

# alloc: Heap Allocation Without std

## Introduction
Sometimes fixed-size buffers are not enough — you need dynamic collections. The `alloc` crate sits between `core` and `std`, providing heap-allocated types like `Vec`, `String`, `Box`, and `Arc` without requiring an operating system. You just need to supply a global allocator.

## Key Concepts
- **`alloc` crate**: Provides heap types (`Vec`, `String`, `Box`, `Rc`, `Arc`) in a no_std environment.
- **`#[global_allocator]`**: A static attribute that registers your chosen allocator implementation.
- **Global allocator**: An implementation of the `GlobalAlloc` trait that manages a region of memory as a heap.

## Real World Context
If your firmware needs to parse variable-length JSON payloads, build dynamic command buffers, or manage a queue of network packets, you need heap allocation. The `alloc` crate gives you familiar Rust collections without pulling in `std`.

## Deep Dive

To use `alloc`, declare it as an external crate and register a global allocator. The following example uses `linked-list-allocator` with a 32 KB static heap:

```rust
#![no_std]

extern crate alloc;

use alloc::vec::Vec;
use alloc::string::String;
use alloc::boxed::Box;

use linked_list_allocator::LockedHeap;

#[global_allocator]
static ALLOCATOR: LockedHeap = LockedHeap::empty();

fn init_heap() {
    const HEAP_SIZE: usize = 1024 * 32; // 32 KB
    static mut HEAP_MEM: [u8; HEAP_SIZE] = [0; HEAP_SIZE];

    unsafe {
        let heap_start = &raw const HEAP_MEM as usize;
        ALLOCATOR.lock().init(heap_start, HEAP_SIZE);
    }
}
```

After calling `init_heap()`, you can use `Vec::new()`, `String::from("hello")`, and other heap types normally. The allocator carves out memory from the static byte array.

Here are common allocator crates for embedded targets:

| Crate | Use Case |
|-------|----------|
| `linked-list-allocator` | Simple, small code size |
| `embedded-alloc` | Cortex-M optimized |
| `dlmalloc` | General purpose, larger |
| `talc` | Fast, low fragmentation |

Note that `wee_alloc` was archived in 2022 and should not be used in new projects.

## Common Pitfalls
1. **Forgetting to call the init function** — The allocator starts empty. If you allocate before initializing, you get a panic or hard fault. Always call your heap init in the entry point before any heap usage.
2. **Heap too small** — A 4 KB heap fills quickly with `Vec` growth. Profile your peak usage and add margin. Consider `heapless` for known-size collections.
3. **Fragmentation on long-running systems** — A linked-list allocator can fragment over time. For production firmware, prefer pool allocators or fixed-size collections.

## Best Practices
1. **Prefer `heapless` when sizes are known** — If your buffer is always 64 bytes, use `heapless::Vec<u8, 64>` instead of `alloc::vec::Vec<u8>`. Zero fragmentation, zero allocation failure at runtime.
2. **Set an alloc error handler** — Define `#[alloc_error_handler]` to handle out-of-memory gracefully (e.g., log and reset) rather than panicking.
3. **Measure peak heap usage** — Use the allocator's stats (many crates provide `used()` / `free()`) during development to right-size your heap.

## Summary
- `alloc` provides `Vec`, `String`, `Box`, `Rc`, and `Arc` in no_std.
- You must register a `#[global_allocator]` and initialize it with a memory region.
- `linked-list-allocator`, `embedded-alloc`, and `talc` are popular choices.
- Prefer `heapless` collections when maximum sizes are known at compile time.

## Code Examples

**heapless collections provide Vec, String, and LinearMap with fixed compile-time capacity — no allocator needed, no fragmentation risk**

```rust
#![no_std]

use heapless::{Vec, String, LinearMap};
use core::fmt::Write;

// All collections are stack-allocated with compile-time capacity
fn process_sensor_data() {
    // Vector with capacity for 16 readings
    let mut readings: Vec<i32, 16> = Vec::new();
    for i in 0..10 {
        readings.push(i * 100).unwrap(); // Fails if capacity exceeded
    }

    // String with capacity for 64 characters
    let mut message: String<64> = String::new();
    write!(message, "Readings: {}", readings.len()).unwrap();

    // Map with capacity for 8 key-value pairs
    let mut config: LinearMap<&str, i32, 8> = LinearMap::new();
    config.insert("timeout_ms", 1000).unwrap();
    config.insert("retries", 3).unwrap();

    if let Some(&timeout) = config.get("timeout_ms") {
        // Use timeout value — guaranteed no heap allocation
    }
}
```


## Resources

- [alloc crate documentation](https://doc.rust-lang.org/alloc/) — API reference for heap-allocated types available without std
- [heapless crate documentation](https://docs.rs/heapless/) — Fixed-capacity collections for no_std environments

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*