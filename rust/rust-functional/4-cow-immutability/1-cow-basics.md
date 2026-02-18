---
source_course: "rust-functional"
source_lesson: "rust-func-cow-basics"
---

# Cow<'a, T>: Clone on Write

Delays cloning until mutation is needed.

```rust
use std::borrow::Cow;

enum Cow<'a, T> where T: ToOwned {
    Borrowed(&'a T),
    Owned(<T as ToOwned>::Owned),
}
```

## Basic Usage

```rust
use std::borrow::Cow;

fn process(input: Cow<str>) -> Cow<str> {
    if input.contains("bad") {
        // Only clone if we need to modify
        let owned = input.into_owned().replace("bad", "good");
        Cow::Owned(owned)
    } else {
        input  // Return borrowed, no clone!
    }
}

let borrowed: Cow<str> = Cow::Borrowed("hello");
let owned: Cow<str> = Cow::Owned(String::from("world"));

let result = process(borrowed);  // No allocation if "hello" returned
```

## Use Cases

### 1. Functions That May or May Not Modify

```rust
fn maybe_uppercase(s: &str, should_upper: bool) -> Cow<str> {
    if should_upper {
        Cow::Owned(s.to_uppercase())
    } else {
        Cow::Borrowed(s)
    }
}
```

### 2. Lazy Parsing

```rust
fn unescape(s: &str) -> Cow<str> {
    if s.contains('\\') {
        // Only allocate if escapes present
        Cow::Owned(s.replace("\\\\", "\\"))
    } else {
        Cow::Borrowed(s)
    }
}
```

### 3. Storing Either Borrowed or Owned

```rust
struct Config<'a> {
    name: Cow<'a, str>,
    values: Vec<Cow<'a, str>>,
}
```

## Methods

```rust
let mut cow = Cow::Borrowed("hello");

// Force to owned
let owned: String = cow.into_owned();

// Get mutable reference (clones if borrowed)
let mut cow2 = Cow::Borrowed("hello");
cow2.to_mut().push_str(" world");  // Clones here
```

## Code Examples

**Cow usage patterns**

```rust
use std::borrow::Cow;

// Efficient string normalization
fn normalize_path(path: &str) -> Cow<str> {
    if path.contains("//") || path.contains("/./") {
        // Need to clean up - allocate
        let cleaned = path
            .replace("//", "/")
            .replace("/./", "/");
        Cow::Owned(cleaned)
    } else {
        // Already clean - just borrow
        Cow::Borrowed(path)
    }
}

// Most paths won't need allocation
let p1 = normalize_path("/usr/local/bin");  // Borrowed
let p2 = normalize_path("/usr//local");     // Owned

// Using Cow in structs
struct ErrorMessage<'a> {
    code: u32,
    message: Cow<'a, str>,
}

impl<'a> ErrorMessage<'a> {
    // Static error (no allocation)
    fn not_found() -> Self {
        ErrorMessage {
            code: 404,
            message: Cow::Borrowed("Not found"),
        }
    }
    
    // Dynamic error (allocates)
    fn custom(code: u32, msg: String) -> Self {
        ErrorMessage {
            code,
            message: Cow::Owned(msg),
        }
    }
}
```


---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*