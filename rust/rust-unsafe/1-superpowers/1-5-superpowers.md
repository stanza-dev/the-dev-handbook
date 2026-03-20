---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-5-superpowers"
---

# The 5 Unsafe Superpowers

## Introduction
Rust's safety guarantees are one of its greatest strengths, but some valid programs cannot be expressed within the safe subset of the language. The `unsafe` keyword unlocks exactly five additional capabilities that the compiler cannot verify. Understanding what these are — and what `unsafe` does NOT disable — is critical for writing correct low-level code.

## Key Concepts
- **Unsafe block**: A block of code where you promise the compiler that you have manually verified the safety invariants.
- **Raw pointer**: A pointer type (`*const T` / `*mut T`) that lacks the aliasing and validity guarantees of references.
- **Unsafe function**: A function whose caller must uphold certain invariants the compiler cannot check.
- **Safety invariant**: A condition that must be true for a piece of unsafe code to be sound.

## Real World Context
Every major Rust crate that does systems programming — `tokio`, `hyper`, `serde`, standard library collections — uses `unsafe` internally to implement efficient data structures. As a working developer, you will encounter `unsafe` in FFI bindings, performance-critical code, and when implementing data structures that the borrow checker cannot express.

## Deep Dive

`unsafe` gives you exactly five additional capabilities:

### 1. Dereference Raw Pointers

Raw pointers can be null, dangling, or unaligned. Dereferencing them requires an `unsafe` block because the compiler cannot verify they point to valid memory.

```rust
let x = 42;
let ptr = &x as *const i32;

unsafe {
    println!("{}", *ptr); // Dereference raw pointer
}
```

The `unsafe` block tells the compiler: "I have verified this pointer is valid, aligned, and points to initialized data."

### 2. Call Unsafe Functions

Some functions have preconditions the compiler cannot check. These are marked `unsafe fn`.

**Important (Edition 2024):** Since Edition 2024 (Rust 1.85), the `unsafe_op_in_unsafe_fn` lint is warn-by-default. The body of an `unsafe fn` is no longer implicitly an unsafe context. You must use explicit `unsafe {}` blocks inside unsafe function bodies:

```rust
/// # Safety
/// `ptr` must be valid, aligned, and point to initialized data.
unsafe fn read_ptr(ptr: *const i32) -> i32 {
    // Edition 2024: explicit unsafe block required inside unsafe fn
    unsafe { *ptr }
}

unsafe {
    let value = read_ptr(&42 as *const i32);
}
```

This change makes it clearer exactly which operations inside an unsafe function are the dangerous ones.

### 3. Access Mutable Statics

**Deprecation Warning (Edition 2024):** Creating references to `static mut` is denied by default in Edition 2024 because it is trivially easy to create aliasing `&mut` references, which is undefined behavior. The recommended replacements are atomic types or `Mutex`.

```rust
// DEPRECATED in Edition 2024 — avoid this pattern
static mut COUNTER: i32 = 0;

unsafe {
    // Creates a reference to static mut — denied by default!
    COUNTER += 1;
}
```

Instead, use atomics or synchronization primitives:

```rust
use std::sync::atomic::{AtomicI32, Ordering};

// RECOMMENDED: Thread-safe, no unsafe needed
static COUNTER: AtomicI32 = AtomicI32::new(0);

fn increment() {
    COUNTER.fetch_add(1, Ordering::Relaxed);
    println!("{}", COUNTER.load(Ordering::Relaxed));
}
```

For more complex state, use `Mutex` or `RwLock` wrapped in `OnceLock`.

### 4. Implement Unsafe Traits

Some traits have invariants that the compiler cannot verify. Implementing them requires `unsafe impl`.

```rust
unsafe trait Guaranteed {
    fn check(&self) -> bool;
}

unsafe impl Guaranteed for MyType {
    fn check(&self) -> bool { true }
}
```

The classic example is `Send` and `Sync` — implementing these incorrectly can cause data races.

### 5. Access Union Fields

Reading a union field reinterprets the bits, which can produce invalid values.

```rust
union IntOrFloat {
    i: i32,
    f: f32,
}

let u = IntOrFloat { i: 42 };
unsafe {
    println!("{}", u.f); // Reinterpret bits as float
}
```

### What unsafe Does NOT Disable

This is crucial to understand:

- The borrow checker still runs
- Type checking still happens
- Lifetime checking still works
- All other safety checks remain

`unsafe` is a contract: "I, the programmer, have verified the invariants the compiler cannot check."

## Common Pitfalls
1. **Assuming unsafe disables the borrow checker** — It does not. You still cannot have two `&mut` references to the same data, even inside `unsafe`. Raw pointers bypass the borrow checker, but references do not.
2. **Writing overly large unsafe blocks** — Keep `unsafe` blocks as small as possible. Wrap them in safe abstractions so the surface area for bugs is minimal.
3. **Forgetting the Edition 2024 changes** — Unsafe function bodies now trigger warnings for implicit unsafe operations, and `static mut` references are denied by default.

## Best Practices
1. **Always add a `// SAFETY:` comment** — Document why each `unsafe` block is sound. This is standard practice in the Rust ecosystem and required by `clippy::undocumented_unsafe_blocks`.
2. **Wrap unsafe in safe APIs** — Expose a safe public interface that upholds invariants internally. Users of your API should never need to write `unsafe`.
3. **Use `unsafe fn` only when callers must uphold invariants** — If you can verify safety inside the function, use a safe function with an internal `unsafe` block instead.

## Summary
- `unsafe` unlocks exactly 5 capabilities: raw pointer derefs, calling unsafe functions, mutable statics, unsafe trait impls, and union field access.
- Edition 2024 warns on implicit unsafe operations inside `unsafe fn` bodies.
- `static mut` is deprecated in Edition 2024; use `AtomicI32`, `Mutex`, or `OnceLock` instead.
- `unsafe` does NOT disable the borrow checker, type checker, or lifetime checker.
- Always document safety invariants with `// SAFETY:` comments.

## Code Examples

**Safe wrappers around unsafe operations — shows the difference between unsafe blocks and unsafe functions, and how to encapsulate unsafe code behind a safe API**

```rust
// unsafe block vs unsafe function

// unsafe BLOCK: "I'm doing something unsafe here"
fn safe_wrapper(ptr: *const i32) -> Option<i32> {
    if ptr.is_null() {
        return None;
    }
    // SAFETY: We checked the pointer is not null
    unsafe { Some(*ptr) }
}

// unsafe FUNCTION: "Caller must uphold invariants"
/// # Safety
/// ptr must be valid and properly aligned
unsafe fn read_ptr(ptr: *const i32) -> i32 {
    // Edition 2024: explicit unsafe block required
    unsafe { *ptr }
}

// Best practice: wrap unsafe in safe APIs
pub struct SafeBox {
    ptr: *mut i32,
}

impl SafeBox {
    pub fn new(value: i32) -> Self {
        let ptr = Box::into_raw(Box::new(value));
        SafeBox { ptr }
    }

    pub fn get(&self) -> i32 {
        // SAFETY: ptr is always valid (from Box::into_raw)
        unsafe { *self.ptr }
    }
}

impl Drop for SafeBox {
    fn drop(&mut self) {
        // SAFETY: ptr came from Box::into_raw
        unsafe { drop(Box::from_raw(self.ptr)); }
    }
}
```

**Edition 2024 replacements for static mut — AtomicI32 for simple counters and OnceLock<Mutex<T>> for complex shared state**

```rust
use std::sync::atomic::{AtomicI32, Ordering};
use std::sync::Mutex;

// RECOMMENDED: Atomic for simple counters
static HIT_COUNT: AtomicI32 = AtomicI32::new(0);

fn record_hit() {
    HIT_COUNT.fetch_add(1, Ordering::Relaxed);
}

fn get_hits() -> i32 {
    HIT_COUNT.load(Ordering::Relaxed)
}

// RECOMMENDED: Mutex for complex state
static CONFIG: std::sync::OnceLock<Mutex<Vec<String>>> = std::sync::OnceLock::new();

fn add_config(value: String) {
    let config = CONFIG.get_or_init(|| Mutex::new(Vec::new()));
    config.lock().unwrap().push(value);
}
```


## Resources

- [Unsafe Rust — The Rust Book](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html) — The Rust Book chapter on unsafe, covering all five superpowers
- [The Rustonomicon — Meet Safe and Unsafe](https://doc.rust-lang.org/nomicon/meet-safe-and-unsafe.html) — Advanced guide to the boundary between safe and unsafe Rust

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*