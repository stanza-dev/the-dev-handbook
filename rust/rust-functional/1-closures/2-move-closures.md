---
source_course: "rust-functional"
source_lesson: "rust-func-move-closures"
---

# Move Closures and Borrowing

## Introduction
By default, closures capture variables using the least restrictive borrowing mode needed. The `move` keyword overrides this default and forces the closure to take ownership of all captured variables. Understanding when to use `move` is essential for threading, returning closures, and async programming.

## Key Concepts
- **Default capturing**: The compiler picks `&T`, `&mut T`, or `T` (move) for each captured variable based on how the closure uses it.
- **`move` keyword**: Forces all captured variables to be moved (or copied, for `Copy` types) into the closure.
- **Copy types with `move`**: For types that implement `Copy` (like `i32`, `bool`, `f64`), `move` copies the value rather than transferring ownership. The original remains usable.

## Real World Context
You will reach for `move` closures in three common scenarios: spawning threads (the closure must own its data because the thread may outlive the caller), returning closures from functions (captured locals would be dangling references without `move`), and async blocks (the future may be polled after the surrounding scope ends).

## Deep Dive

### Default capture behavior

Without `move`, Rust borrows by the least powerful mode needed:

```rust
let name = String::from("Alice");

let greet = || println!("Hello, {name}"); // Borrows &name

greet();
println!("Still valid: {name}"); // name is still accessible
```

The closure only needs to read `name`, so it captures an immutable reference. The original `name` remains usable after the closure.

### Forcing ownership with `move`

The `move` keyword transfers ownership into the closure:

```rust
let name = String::from("Alice");

let greet = move || println!("Hello, {name}"); // Owns name now

greet();
// println!("{name}"); // Error: name was moved into the closure
```

After the `move`, the outer scope no longer owns `name`.

### Thread spawning requires `move`

Spawned threads may outlive the scope that created them, so they must own their data:

```rust
use std::thread;

let data = vec![1, 2, 3, 4, 5];

let handle = thread::spawn(move || {
    let sum: i32 = data.iter().sum();
    println!("Sum: {sum}"); // Output: Sum: 15
});

handle.join().unwrap();
// data is no longer accessible here
```

Without `move`, the compiler would reject this code because `data` could be dropped before the thread finishes.

### Returning closures from functions

When a function returns a closure, any captured locals would be destroyed at the end of the function. `move` ensures the closure owns them:

```rust
fn make_adder(base: i32) -> impl Fn(i32) -> i32 {
    move |x| x + base // base is moved (copied, since i32 is Copy)
}

let add_ten = make_adder(10);
assert_eq!(add_ten(5), 15);
assert_eq!(add_ten(20), 30);
```

Since `i32` implements `Copy`, `base` is copied into the closure rather than moved.

### Move with Copy types

For `Copy` types, `move` copies rather than moves — the original remains valid:

```rust
let threshold = 42; // i32 is Copy

let check = move || threshold > 0; // threshold is copied

println!("Original still valid: {threshold}"); // Works fine
```

This is an important distinction. `move` does not always invalidate the original binding.

## Common Pitfalls
1. **Using `move` when a borrow would suffice** — `move` transfers ownership of *all* captured variables. If you only need to move one variable, consider cloning the others first and using `move` on the clone.
2. **Expecting `move` to deep-clone** — `move` transfers ownership; it does not clone. If you need the original value after the closure, clone before the closure.
3. **Forgetting that `Copy` types survive `move`** — Integers, booleans, and other `Copy` types are copied, not moved. The original is still valid.

## Best Practices
1. **Clone-then-move for selective ownership** — When only some captured variables need to be owned, clone the ones you want to keep and let `move` take the clones.
2. **Always use `move` with `thread::spawn`** — The compiler usually requires it anyway, but being explicit documents intent.
3. **Prefer `move` for `async` blocks** — Async blocks often outlive their enclosing scope. Using `move` prevents subtle lifetime errors.

## Summary
- Default closures borrow by the least restrictive mode needed.
- `move` forces the closure to take ownership of all captures.
- `Copy` types are copied, not moved — the original remains valid.
- `move` is essential for threads, returned closures, and async blocks.
- Rust 1.94 makes per-field closure capturing more precise, reducing unnecessary moves.

## Code Examples

**Clone-then-move for threads and returning stateful closures — two of the most common move patterns in production Rust**

```rust
use std::thread;

// Clone-then-move pattern for selective ownership
fn parallel_search(haystack: &[String], needle: String) -> bool {
    let mid = haystack.len() / 2;
    let (left, right) = haystack.split_at(mid);

    // Clone data for the spawned thread
    let left_owned = left.to_vec();
    let needle_clone = needle.clone();

    let handle = thread::spawn(move || {
        left_owned.iter().any(|item| item.contains(&needle_clone))
    });

    // Main thread searches the right half with the original needle
    let right_found = right.iter().any(|item| item.contains(&needle));
    let left_found = handle.join().unwrap();

    left_found || right_found
}

// Returning a closure that owns its state
fn make_counter(start: u32) -> impl FnMut() -> u32 {
    let mut current = start;
    move || {
        let value = current;
        current += 1;
        value
    }
}

fn main() {
    let mut counter = make_counter(10);
    assert_eq!(counter(), 10);
    assert_eq!(counter(), 11);
    assert_eq!(counter(), 12);
}
```


## Resources

- [Closures: Capturing References or Moving Ownership](https://doc.rust-lang.org/book/ch13-01-closures.html#capturing-references-or-moving-ownership) — Official Rust Book section on move closures and capture semantics

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*