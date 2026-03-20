---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-global-allocators"
---

# Custom Global Allocators

## Introduction
Rust lets you replace the default memory allocator with a custom one using the `#[global_allocator]` attribute. This is critical for embedded systems (which may not have a standard allocator), game engines (which need arena allocators for performance), and any application where allocation behavior must be controlled precisely.

## Key Concepts
- **`GlobalAlloc` trait**: The trait that custom allocators must implement: `alloc`, `dealloc`, and optionally `realloc`.
- **`#[global_allocator]`**: Attribute marking a static value as the global allocator for the program.
- **`Layout`**: Describes the size and alignment requirements of an allocation.
- **Arena allocator**: An allocator that allocates from a fixed-size buffer and frees all memory at once.

## Real World Context
Firefox uses `jemalloc` via a custom global allocator for better multithreaded performance. Embedded systems use bump allocators. Game engines use per-frame arena allocators that reset every frame. The `mimalloc` and `jemalloc` crates are popular drop-in replacements.

## Deep Dive

### Using a Third-Party Allocator

The simplest way to change the allocator is to use a crate like `mimalloc`:

```rust
use mimalloc::MiMalloc;

#[global_allocator]
static GLOBAL: MiMalloc = MiMalloc;

fn main() {
    // All heap allocations now use mimalloc
    let v = vec![1, 2, 3];
}
```

This single change can improve allocation-heavy programs by 10-30%.

### Implementing `GlobalAlloc`

To write your own allocator, implement the `GlobalAlloc` trait:

```rust
use std::alloc::{GlobalAlloc, Layout};
use std::sync::atomic::{AtomicUsize, Ordering};

struct CountingAllocator {
    alloc_count: AtomicUsize,
    dealloc_count: AtomicUsize,
}

unsafe impl GlobalAlloc for CountingAllocator {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
        self.alloc_count.fetch_add(1, Ordering::Relaxed);
        // Delegate to the system allocator
        unsafe { std::alloc::System.alloc(layout) }
    }

    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout) {
        self.dealloc_count.fetch_add(1, Ordering::Relaxed);
        unsafe { std::alloc::System.dealloc(ptr, layout) }
    }
}

#[global_allocator]
static ALLOCATOR: CountingAllocator = CountingAllocator {
    alloc_count: AtomicUsize::new(0),
    dealloc_count: AtomicUsize::new(0),
};
```

This wraps the system allocator to count allocations — useful for detecting leaks.

### Bump Allocator for Embedded

A bump allocator hands out memory from a fixed buffer:

```rust
use std::alloc::{GlobalAlloc, Layout};
use std::sync::atomic::{AtomicUsize, Ordering};

struct BumpAllocator {
    heap: [u8; 4096],
    offset: AtomicUsize,
}

unsafe impl GlobalAlloc for BumpAllocator {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
        let align = layout.align();
        let size = layout.size();

        loop {
            let current = self.offset.load(Ordering::Relaxed);
            let aligned = (current + align - 1) & !(align - 1);
            let new_offset = aligned + size;

            if new_offset > self.heap.len() {
                return std::ptr::null_mut(); // Out of memory
            }

            if self.offset.compare_exchange_weak(
                current, new_offset, Ordering::AcqRel, Ordering::Relaxed
            ).is_ok() {
                return self.heap.as_ptr().add(aligned) as *mut u8;
            }
        }
    }

    unsafe fn dealloc(&self, _ptr: *mut u8, _layout: Layout) {
        // Bump allocators don't free individual allocations
    }
}
```

Bump allocators are extremely fast (just increment a pointer) but cannot free individual allocations.

## Common Pitfalls
1. **Returning unaligned pointers** — The returned pointer must satisfy `layout.align()`. Forgetting alignment causes UB.
2. **Allocating inside the allocator** — If your allocator implementation allocates (e.g., using `Vec`), it causes infinite recursion.
3. **Not handling zero-size allocations** — `Layout` guarantees size > 0, but if you implement `Allocator` (nightly), you must handle zero-size types.

## Best Practices
1. **Use established allocators for production** — `mimalloc`, `jemalloc`, and `snmalloc` are well-tested.
2. **Wrap the system allocator for debugging** — A counting/logging wrapper helps find allocation hotspots.
3. **Test with ASAN/MSAN** — Address sanitizer and memory sanitizer catch allocator bugs that are invisible in normal testing.

## Summary
- `#[global_allocator]` replaces the program's default allocator.
- Implement `GlobalAlloc` with `alloc` and `dealloc` methods.
- Third-party allocators like `mimalloc` are drop-in replacements.
- Bump allocators are fast but cannot free individual allocations.
- Always respect `Layout` alignment requirements in custom allocators.

## Code Examples

**A tracking allocator that monitors total heap usage — wraps the system allocator to count bytes allocated and deallocated**

```rust
use std::alloc::{GlobalAlloc, Layout, System};
use std::sync::atomic::{AtomicUsize, Ordering};

/// An allocator that tracks total allocated bytes.
struct TrackingAllocator {
    bytes_allocated: AtomicUsize,
}

unsafe impl GlobalAlloc for TrackingAllocator {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
        self.bytes_allocated.fetch_add(layout.size(), Ordering::Relaxed);
        unsafe { System.alloc(layout) }
    }

    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout) {
        self.bytes_allocated.fetch_sub(layout.size(), Ordering::Relaxed);
        unsafe { System.dealloc(ptr, layout) }
    }
}

#[global_allocator]
static ALLOC: TrackingAllocator = TrackingAllocator {
    bytes_allocated: AtomicUsize::new(0),
};

fn main() {
    let v: Vec<u8> = Vec::with_capacity(1024);
    println!("Bytes allocated: {}", ALLOC.bytes_allocated.load(Ordering::Relaxed));
    drop(v);
    println!("After drop: {}", ALLOC.bytes_allocated.load(Ordering::Relaxed));
}
```


## Resources

- [GlobalAlloc trait — std documentation](https://doc.rust-lang.org/std/alloc/trait.GlobalAlloc.html) — API reference for the GlobalAlloc trait and #[global_allocator]

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*