---
source_course: "rust-embedded"
source_lesson: "rust-embedded-alloc-crate"
---

# alloc: Heap Without std

The `alloc` crate provides heap-allocated types without full std.

```rust
#![no_std]

extern crate alloc;

use alloc::vec::Vec;
use alloc::string::String;
use alloc::boxed::Box;
use alloc::rc::Rc;
use alloc::sync::Arc;
```

## Requires a Global Allocator

```rust
use linked_list_allocator::LockedHeap;

#[global_allocator]
static ALLOCATOR: LockedHeap = LockedHeap::empty();

fn init_heap() {
    const HEAP_SIZE: usize = 1024 * 32; // 32KB
    static mut HEAP: [u8; HEAP_SIZE] = [0; HEAP_SIZE];
    
    unsafe {
        ALLOCATOR.lock().init(
            HEAP.as_ptr() as usize,
            HEAP_SIZE
        );
    }
}
```

## Common Allocators

| Crate | Use Case |
|-------|----------|
| `linked-list-allocator` | Simple, small |
| `embedded-alloc` | Cortex-M optimized |
| `dlmalloc` | General purpose |
| `wee_alloc` | Minimal size (WASM) |

## Avoiding Heap: heapless

```rust
use heapless::Vec;
use heapless::String;

// Fixed-capacity, stack-allocated
let mut v: Vec<i32, 8> = Vec::new();  // Max 8 elements
v.push(1).unwrap();
v.push(2).unwrap();

let mut s: String<16> = String::new();  // Max 16 chars
s.push_str("hello").unwrap();
```

## heapless Types

- `Vec<T, N>` - Fixed-capacity vector
- `String<N>` - Fixed-capacity string
- `LinearMap<K, V, N>` - Fixed-capacity map
- `spsc::Queue<T, N>` - Lock-free queue
- `pool::Pool<T>` - Memory pool

## Code Examples

**heapless collections**

```rust
#![no_std]

use heapless::{Vec, String, LinearMap};

// Stack-allocated collections
fn example() {
    // Vector with capacity 16
    let mut numbers: Vec<i32, 16> = Vec::new();
    for i in 0..10 {
        numbers.push(i).unwrap();
    }
    
    // String with capacity 64
    let mut message: String<64> = String::new();
    write!(message, "Count: {}", numbers.len()).unwrap();
    
    // Map with capacity 8
    let mut config: LinearMap<&str, i32, 8> = LinearMap::new();
    config.insert("timeout", 1000).unwrap();
    config.insert("retries", 3).unwrap();
    
    // Get value
    if let Some(timeout) = config.get("timeout") {
        // Use timeout
    }
}

// Producer-consumer queue (interrupt-safe)
use heapless::spsc::Queue;

static mut QUEUE: Queue<u8, 32> = Queue::new();

fn producer() {
    let mut producer = unsafe { QUEUE.split().0 };
    producer.enqueue(42).ok();
}

fn consumer() {
    let mut consumer = unsafe { QUEUE.split().1 };
    while let Some(value) = consumer.dequeue() {
        // Process value
    }
}
```


---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*