---
source_course: "rust"
source_lesson: "rust-derive-common-traits"
---

# The derive Attribute

The `#[derive]` attribute automatically implements common traits.

```rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
```

## Common Derivable Traits

| Trait | Purpose | Example |
|-------|---------|--------|
| `Debug` | Format with `{:?}` | Debugging output |
| `Clone` | Explicit `.clone()` | Deep copying |
| `Copy` | Implicit copy | Simple types |
| `PartialEq` | `==` and `!=` | Equality comparison |
| `Eq` | Total equality | Hash map keys |
| `PartialOrd` | `<`, `>`, `<=`, `>=` | Partial ordering |
| `Ord` | Total ordering | Sorting |
| `Hash` | Hashing | HashMap keys |
| `Default` | `Default::default()` | Default values |

## Copy Requires Clone

```rust
#[derive(Debug, Clone, Copy)] // Copy requires Clone
struct Point {
    x: i32,
    y: i32,
}

let p1 = Point { x: 1, y: 2 };
let p2 = p1; // Copy, not move!
println!("{:?} {:?}", p1, p2); // Both valid
```

## Default Trait

```rust
#[derive(Default, Debug)]
struct Config {
    debug: bool,      // false
    port: u16,        // 0
    name: String,     // ""
}

let config = Config::default();
let config = Config {
    debug: true,
    ..Default::default() // Fill rest with defaults
};
```

See [Derivable Traits](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html).

## Code Examples

**HashMap with custom key type**

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
struct UserId(u64);

use std::collections::HashMap;

let mut users = HashMap::new();
users.insert(UserId(1), "Alice");
users.insert(UserId(2), "Bob");

// Works because UserId implements Hash + Eq
if let Some(name) = users.get(&UserId(1)) {
    println!("Found: {name}");
}
```


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*