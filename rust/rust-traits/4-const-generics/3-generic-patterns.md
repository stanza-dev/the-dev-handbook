---
source_course: "rust-traits"
source_lesson: "rust-traits-const-generic-patterns"
---

# Practical Const Generic Patterns

## Introduction
Const generics enable several powerful patterns: type-level integers for dimensional analysis, fixed-capacity data structures, and compile-time size validation. This lesson covers real-world patterns you will encounter in production Rust code.

## Key Concepts
- **Type-Level Integers**: Using const generics to encode numeric properties in the type system.
- **Fixed-Capacity Structures**: Data structures whose maximum size is a const generic, avoiding heap allocation.
- **Compile-Time Assertions**: Using const generics to enforce invariants at compile time.

## Real World Context
Embedded Rust uses fixed-size buffers (`heapless::Vec<T, N>`) to avoid heap allocation. The `nalgebra` crate uses const generics for matrix dimensions. Network protocols use const generics to define packet structures with known sizes.

## Deep Dive

### Fixed-Capacity Stack Vec

```rust
struct StackVec<T, const N: usize> {
    data: [std::mem::MaybeUninit<T>; N],
    len: usize,
}

impl<T, const N: usize> StackVec<T, N> {
    fn new() -> Self {
        StackVec {
            data: std::array::from_fn(|_| std::mem::MaybeUninit::uninit()),
            len: 0,
        }
    }

    fn push(&mut self, value: T) -> Result<(), T> {
        if self.len >= N {
            return Err(value); // Compile-time capacity!
        }
        self.data[self.len] = std::mem::MaybeUninit::new(value);
        self.len += 1;
        Ok(())
    }
}
```

### Bool Const Generics for Feature Flags

```rust
struct Logger<const VERBOSE: bool>;

impl Logger<true> {
    fn log(&self, msg: &str) {
        println!("[VERBOSE] {msg}");
    }
}

impl Logger<false> {
    fn log(&self, _msg: &str) {
        // No-op in non-verbose mode
    }
}
```

The compiler eliminates the non-verbose path entirely — zero runtime cost for disabled logging.

### Array Initialization with `std::array::from_fn`

`std::array::from_fn` uses const generics to create arrays of any size from a closure:

```rust
// Create an array of N elements using a closure
let squares: [i32; 5] = std::array::from_fn(|i| (i * i) as i32);
assert_eq!(squares, [0, 1, 4, 9, 16]);

// Works with any const generic size
fn make_identity<const N: usize>() -> [[f64; N]; N] {
    std::array::from_fn(|i| std::array::from_fn(|j| if i == j { 1.0 } else { 0.0 }))
}
let m: [[f64; 3]; 3] = make_identity();
```

## Common Pitfalls
1. **Stack overflow with large const generics** — `StackVec<u8, 1_000_000>` allocates 1 MB on the stack. Use heap allocation for large buffers.
2. **Forgetting that each `N` value is a different type** — `Buffer<64>` and `Buffer<128>` are completely different types and cannot be mixed.

## Best Practices
1. **Use const generics for embedded and no-std code** — Fixed-capacity structures avoid heap allocation entirely.
2. **Combine with traits for size-independent APIs** — Implement traits on `Buffer<N>` for any `N` so that generic code can work with any size.

## Summary
- Const generics enable fixed-capacity, stack-allocated data structures.
- Bool const generics can serve as compile-time feature flags with zero runtime cost.
- Each distinct const value produces a different type.
- `std::array::from_fn` demonstrates const generics in the standard library for array initialization.

## Code Examples

**Bool const generics create separate implementations for encrypted and unencrypted connections, with zero runtime branching — the unused path is eliminated at compile time**

```rust
// Bool const generic for compile-time feature toggling
struct Connection<const ENCRYPTED: bool>;

impl Connection<true> {
    fn send(&self, data: &[u8]) {
        let encrypted = encrypt(data);
        transmit(&encrypted);
    }
}

impl Connection<false> {
    fn send(&self, data: &[u8]) {
        transmit(data); // No encryption overhead
    }
}

// The compiler generates only the code path you use
fn encrypt(data: &[u8]) -> Vec<u8> { todo!() }
fn transmit(data: &[u8]) { todo!() }
```


## Resources

- [Const Generics in Practice](https://doc.rust-lang.org/reference/items/generics.html#const-generics) — Rust Reference on const generics and their usage patterns

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*