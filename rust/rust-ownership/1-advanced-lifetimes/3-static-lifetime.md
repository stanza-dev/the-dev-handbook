---
source_course: "rust-ownership"
source_lesson: "rust-ownership-static-lifetime"
---

# The 'static Lifetime

`'static` means the reference can live for the entire program duration.

## String Literals

All string literals are `'static`:

```rust
let s: &'static str = "I live forever!";
// Stored in binary, never freed
```

## Static Variables

```rust
static HELLO: &str = "Hello, world!";
// Also 'static
```

## Common Misconception

`'static` doesn't mean "lives forever" - it means **CAN** live forever:

```rust
// T: 'static means T contains no non-static references
// It doesn't mean T is immortal!
fn spawn<T: Send + 'static>(t: T) { }

let owned = String::from("hello");
spawn(owned);  // String is 'static (owns its data)
// owned is moved, not immortal!
```

## 'static in Trait Bounds

```rust
// T: 'static means T owns all its data
// (or contains only 'static references)
fn needs_static<T: 'static>(t: T) {
    // t can be stored indefinitely
}

needs_static(String::from("owned"));  // OK
// needs_static(&local);  // Error! &local isn't 'static
```

## Leaking to 'static

```rust
let leaked: &'static str = Box::leak(String::from("leaked").into_boxed_str());
// Intentional memory leak - now truly 'static
```

## Code Examples

**Why thread::spawn needs 'static**

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


---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*