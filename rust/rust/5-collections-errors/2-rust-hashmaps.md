---
source_course: "rust"
source_lesson: "rust-hashmaps"
---

# HashMap<K, V>

Key-value storage with O(1) average lookup.

```rust
use std::collections::HashMap;

// Creating
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

// From iterators
let teams = vec!["Blue", "Yellow"];
let initial_scores = vec![10, 50];
let scores: HashMap<_, _> = teams.into_iter().zip(initial_scores).collect();
```

## Accessing Values

```rust
let score = scores.get("Blue"); // Option<&V>

if let Some(s) = scores.get("Blue") {
    println!("Blue: {s}");
}

// Iterate
for (key, value) in &scores {
    println!("{key}: {value}");
}
```

## Ownership

```rust
let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are MOVED and invalid now
```

## Update Patterns

```rust
// Overwrite
scores.insert("Blue".to_string(), 25);

// Insert only if key doesn't exist
scores.entry("Blue".to_string()).or_insert(50);

// Update based on old value
let text = "hello world wonderful world";
let mut map = HashMap::new();
for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}
```

See [Storing Keys with Associated Values in Hash Maps](https://doc.rust-lang.org/stable/book/ch08-03-hash-maps.html).

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


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*