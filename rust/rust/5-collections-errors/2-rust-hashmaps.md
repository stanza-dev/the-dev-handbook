---
source_course: "rust"
source_lesson: "rust-hashmaps"
---

# HashMaps: Key-Value Storage

## Introduction

When you need to associate keys with values and look them up efficiently, Rust provides `HashMap<K, V>`. It offers O(1) average-case lookups and is the go-to data structure for caches, counters, and configuration stores.

## Key Concepts

- **HashMap<K, V>**: A hash table mapping keys of type `K` to values of type `V`. Keys must implement `Eq` and `Hash`.
- **Entry API**: A pattern for conditionally inserting or updating values, avoiding redundant lookups.
- **Ownership transfer**: When you insert owned types like `String` into a HashMap, the map takes ownership of them.

## Real World Context

HashMaps are used in virtually every non-trivial program: counting word frequencies in a document, caching API responses, grouping records by category, or building an in-memory index. The Entry API is particularly valuable in production code because it handles insert-or-update logic in a single, efficient call.

## Deep Dive

You must bring HashMap into scope with a `use` statement, then create and populate it:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

Accessing values uses `.get()`, which returns `Option<&V>` because the key may not exist:

```rust
if let Some(score) = scores.get("Blue") {
    println!("Blue team: {score}");
}
```

Ownership is important: inserting a `String` key moves it into the map. After insertion, the original variable is no longer valid:

```rust
let key = String::from("color");
scores.insert(key, 42);
// key is now invalid — ownership moved to the map
```

The Entry API provides three patterns for safe, efficient updates:

```rust
// Insert only if key is absent
scores.entry("Red".to_string()).or_insert(0);

// Update based on existing value (word counter)
let text = "hello world hello";
let mut counts = HashMap::new();
for word in text.split_whitespace() {
    let count = counts.entry(word).or_insert(0);
    *count += 1;
}
```

The `or_insert` method returns a mutable reference to the value, letting you update it in place.

## Common Pitfalls

1. **Forgetting to import** — Unlike `Vec`, `HashMap` is not in the prelude. You must write `use std::collections::HashMap;` or the compiler will report an unresolved type.
2. **Overwriting with insert** — Calling `.insert()` with an existing key silently replaces the old value. Use the Entry API (`.entry().or_insert()`) when you want insert-if-absent semantics.

## Best Practices

1. **Use the Entry API for insert-or-update** — It performs a single lookup instead of a `.get()` followed by `.insert()`, which would require two lookups and borrowing gymnastics.
2. **Borrow keys for lookups** — The `.get()` method accepts `&Q` where `K: Borrow<Q>`, so you can look up a `String` key with a `&str` slice. No need to allocate a `String` just to search.

## Summary

- `HashMap<K, V>` provides O(1) average-case key-value lookups.
- Keys must implement `Eq + Hash`; inserting owned values transfers ownership.
- `.get()` returns `Option<&V>`, making missing-key handling explicit.
- The Entry API (`entry().or_insert()`) is the idiomatic way to insert or update.
- Always import with `use std::collections::HashMap;`.

## Code Examples

**Word frequency counter**

```rust
use std::collections::HashMap;

fn word_count(text: &str) -> HashMap<&str, i32> {
    let mut counts = HashMap::new();
    for word in text.split_whitespace() {
        *counts.entry(word).or_insert(0) += 1;
    }
    counts
}

let counts = word_count("hello world hello");
// {"hello": 2, "world": 1}
```


## Resources

- [Storing Keys with Associated Values in Hash Maps](https://doc.rust-lang.org/stable/book/ch08-03-hash-maps.html) — Official Rust Book chapter covering HashMap creation, access, and update patterns
- [HashMap API Reference](https://doc.rust-lang.org/std/collections/struct.HashMap.html) — Complete standard library documentation for HashMap<K, V>

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*