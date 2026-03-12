---
source_course: "rust-ownership"
source_lesson: "rust-ownership-static-lifetime"
---

# The 'static Lifetime

## Introduction
The `'static` lifetime is the longest possible lifetime in Rust — it means a reference can be valid for the entire duration of the program. Understanding `'static` is critical because it appears in thread spawning, trait objects, error handling, and many library APIs.

## Key Concepts
- **`'static` reference**: A reference that is valid for the entire program execution, such as string literals embedded in the binary.
- **`T: 'static` bound**: A constraint meaning T owns all its data (or only contains `'static` references), so it can be stored indefinitely.
- **`Box::leak`**: A function that intentionally leaks heap memory to produce a `'static` reference.

## Real World Context
When you call `thread::spawn`, the closure must be `'static` because the spawned thread might outlive the scope that created it. When you use `Box<dyn Error>`, the trait object usually needs a `'static` bound. Every production Rust program encounters `'static` in these common patterns.

## Deep Dive
All string literals have the `'static` lifetime because they are embedded directly in the compiled binary and never freed:

```rust
let s: &'static str = "I live forever!";
// Stored in the binary's read-only data section
```

The most common misconception is that `T: 'static` means the value lives forever. It actually means the value contains no non-static references, so it *could* be kept alive indefinitely:

```rust
// T: 'static means T owns all its data
fn spawn<T: Send + 'static>(t: T) { }

let owned = String::from("hello");
spawn(owned);  // String is 'static (owns its data)
// owned is moved, not immortal!
```

A `String` satisfies `'static` because it owns its heap allocation. A `&'a str` with a non-static lifetime does not, because it borrows data that might be freed.

You can intentionally leak memory to create `'static` references, which is sometimes useful for configuration that lives for the entire program:

```rust
let leaked: &'static str = Box::leak(
    String::from("leaked").into_boxed_str()
);
// Intentional memory leak — now truly 'static
```

## Common Pitfalls
1. **Thinking `'static` means immortal** — A `String` is `'static` but gets dropped normally when it goes out of scope. The bound means it *can* live forever, not that it *will*.
2. **Trying to send references to spawned threads** — Local references are not `'static`, so they cannot cross thread boundaries. Move owned data instead.

## Best Practices
1. **Move owned data into closures for thread spawning** — Use `move ||` to transfer ownership instead of trying to extend reference lifetimes.
2. **Use `'static` bounds sparingly in your own APIs** — Only require `'static` when you genuinely need to store the value indefinitely (e.g., in a long-lived cache or spawned task).

## Summary
- `'static` means a reference or value can be valid for the entire program.
- String literals are `'static` because they live in the binary.
- `T: 'static` means T owns all its data, not that T lives forever.
- `Box::leak` can create `'static` references by intentionally leaking memory.
- Thread spawning requires `'static` because the thread may outlive the caller.

## Code Examples

**Why thread::spawn needs 'static — owned data can be moved, but local references cannot cross thread boundaries**

```rust
// Thread spawning requires 'static
use std::thread;

let local = String::from("hello");

// This works - moves owned data
thread::spawn(move || {
    println!("{local}");
});

// This doesn't work - can't send reference
// let r = &local;
// thread::spawn(move || println!("{r}")); // Error!
```


## Resources

- [The Rust Reference: Static Items](https://doc.rust-lang.org/reference/items/static-items.html) — Official reference for static items and 'static lifetime

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*