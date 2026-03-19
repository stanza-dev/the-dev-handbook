---
source_course: "rust-functional"
source_lesson: "rust-func-cow-basics"
---

# Cow: Clone on Write

## Introduction
`Cow<'a, T>` (Clone on Write) is a smart pointer that can hold either a borrowed reference or an owned value. It delays cloning until mutation is actually needed, making it a powerful tool for functions that usually return borrowed data but occasionally need to allocate. If you have ever written a function that returns `&str` sometimes and `String` other times, `Cow` is the solution.

## Key Concepts
- **`Cow::Borrowed(&T)`**: Wraps a reference. No allocation occurs.
- **`Cow::Owned(T::Owned)`**: Wraps an owned value. Allocation has already occurred.
- **`to_mut()`**: Returns a mutable reference. If the Cow is currently Borrowed, it clones the data first, converting to Owned.
- **`into_owned()`**: Consumes the Cow and returns the owned value. Clones if Borrowed; moves if already Owned.

## Real World Context
`Cow` is widely used in string processing, serialization libraries, and configuration parsers. For example, a URL normalization function usually returns the input unchanged (Borrowed), but occasionally must percent-encode special characters (Owned). Without `Cow`, you would either always allocate a new String or use complex lifetime gymnastics.

## Deep Dive

### The Cow enum

Under the hood, `Cow` is straightforward:

```rust
use std::borrow::Cow;

// Simplified definition:
// enum Cow<'a, B: ToOwned + ?Sized> {
//     Borrowed(&'a B),
//     Owned(<B as ToOwned>::Owned),
// }
```

For strings, `Cow<'a, str>` holds either `&'a str` (borrowed) or `String` (owned).

### Basic usage: avoid allocation when possible

```rust
use std::borrow::Cow;

fn normalize_whitespace(input: &str) -> Cow<str> {
    if input.contains("  ") {
        // Multiple spaces found — must allocate a new string
        let cleaned = input.split_whitespace().collect::<Vec<_>>().join(" ");
        Cow::Owned(cleaned)
    } else {
        // Already clean — just borrow the original
        Cow::Borrowed(input)
    }
}

let clean = normalize_whitespace("hello world"); // Borrowed — no allocation
let dirty = normalize_whitespace("hello  world"); // Owned — allocated
```

Most strings in a typical workload are already clean, so `Cow` avoids allocation in the common case.

### to_mut: clone on first write

The `to_mut()` method gives you a mutable reference, cloning only if the data is currently borrowed:

```rust
use std::borrow::Cow;

let mut message: Cow<str> = Cow::Borrowed("hello");

// This clones "hello" into a String, then appends
message.to_mut().push_str(" world");

assert_eq!(message, "hello world");
// message is now Cow::Owned(String::from("hello world"))
```

If `message` were already `Cow::Owned`, `to_mut()` would return a mutable reference without cloning.

### into_owned: extract the owned value

```rust
use std::borrow::Cow;

let borrowed: Cow<str> = Cow::Borrowed("hello");
let owned_string: String = borrowed.into_owned(); // Clones

let already_owned: Cow<str> = Cow::Owned(String::from("world"));
let moved_string: String = already_owned.into_owned(); // Moves, no clone
```

`into_owned` is useful when you need a `String` (or `Vec<T>`) and want to avoid unnecessary allocation.

### Cow in struct fields

`Cow` works well in structs that may own or borrow their data:

```rust
use std::borrow::Cow;

struct LogEntry<'a> {
    level: &'static str,
    message: Cow<'a, str>,
}

impl<'a> LogEntry<'a> {
    fn info(msg: &'a str) -> Self {
        LogEntry { level: "INFO", message: Cow::Borrowed(msg) }
    }

    fn error(msg: String) -> Self {
        LogEntry { level: "ERROR", message: Cow::Owned(msg) }
    }
}
```

Static log messages use `Borrowed` (zero cost), while dynamically constructed messages use `Owned`.

## Common Pitfalls
1. **Always using `into_owned()` immediately** — If you always convert to an owned value right away, you lose the benefit of Cow. Let the Cow propagate through your code.
2. **Overusing Cow for simple cases** — If your function always needs to allocate, just return `String`. Cow adds complexity that is only justified when borrowing is the common case.
3. **Forgetting the lifetime parameter** — `Cow<str>` requires a lifetime: `Cow<'a, str>` or `Cow<'static, str>` for string literals.

## Best Practices
1. **Use Cow when most calls return borrowed data** — If your function returns a modified value less than ~20% of the time, Cow saves allocations for the common path.
2. **Accept `impl Into<Cow<str>>` for flexible APIs** — This lets callers pass `&str`, `String`, or `Cow<str>` seamlessly.
3. **Prefer `Cow<'static, str>` for error messages** — Static strings are zero-cost, and dynamic formatting produces an Owned variant.

## Summary
- `Cow` delays cloning until mutation or ownership is needed.
- `Cow::Borrowed` wraps a reference with zero allocation.
- `to_mut()` clones on first write; `into_owned()` extracts the owned value.
- Use Cow when most calls can avoid allocation.
- Cow is widely used for string processing, configuration, and error messages.

## Code Examples

**Path normalization that avoids allocation in the common case, and Cow-based error messages with zero-cost static strings**

```rust
use std::borrow::Cow;

// Efficient path normalization — avoids allocation for clean paths
fn normalize_path(path: &str) -> Cow<str> {
    if path.contains("//") || path.contains("/./") {
        let cleaned = path.replace("//", "/").replace("/./", "/");
        Cow::Owned(cleaned)
    } else {
        Cow::Borrowed(path)
    }
}

// Most paths are already clean — no allocation needed
let p1 = normalize_path("/usr/local/bin");   // Cow::Borrowed
let p2 = normalize_path("/usr//local/./bin"); // Cow::Owned

// Cow in error messages
struct AppError<'a> {
    code: u16,
    message: Cow<'a, str>,
}

impl AppError<'static> {
    fn not_found() -> Self {
        AppError { code: 404, message: Cow::Borrowed("Not found") }
    }
    fn custom(code: u16, msg: String) -> Self {
        AppError { code, message: Cow::Owned(msg) }
    }
}
```


## Resources

- [std::borrow::Cow documentation](https://doc.rust-lang.org/std/borrow/enum.Cow.html) — Complete API reference for Cow with method documentation and examples

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*