---
source_course: "rust"
source_lesson: "rust-derive-common-traits"
---

# Deriving Common Traits

## Introduction

Manually implementing common traits like `Debug`, `Clone`, and `PartialEq` for every struct would be tedious and error-prone. Rust's `#[derive]` attribute macro generates these implementations automatically, and it is one of the most frequently used features in the language.

## Key Concepts

- **#[derive(...)]**: An attribute macro placed above a struct or enum definition. It tells the compiler to auto-generate trait implementations based on the struct's fields.
- **Derivable traits**: Traits whose implementations can be mechanically generated. The standard library provides about a dozen, including `Debug`, `Clone`, `Copy`, `PartialEq`, `Eq`, `Hash`, `Default`, `PartialOrd`, and `Ord`.
- **Copy semantics**: Types that implement `Copy` are duplicated implicitly on assignment instead of being moved. `Copy` requires `Clone` and only works for types whose fields are all `Copy` (no heap data).

## Real World Context

Almost every struct in a real Rust codebase has at least `#[derive(Debug)]`. Data transfer objects typically derive `Debug, Clone, PartialEq`. Types used as HashMap keys need `Eq + Hash`. The `serde` library extends derive with `Serialize` and `Deserialize`, making JSON/YAML parsing a one-liner.

## Deep Dive

You apply `derive` by listing the traits in the attribute:

```rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: f64,
    y: f64,
}
```

Now `Point` can be printed with `{:?}`, cloned with `.clone()`, and compared with `==`.

The `Copy` trait enables implicit copies instead of moves. It requires `Clone` and only works when all fields are `Copy`:

```rust
#[derive(Debug, Clone, Copy)]
struct Pixel { r: u8, g: u8, b: u8 }

let p1 = Pixel { r: 255, g: 0, b: 0 };
let p2 = p1; // Copy, not move!
println!("{:?} {:?}", p1, p2); // Both valid
```

A struct with `String` fields cannot derive `Copy` because `String` is heap-allocated.

The `Default` trait provides a `::default()` constructor, combinable with struct update syntax:

```rust
#[derive(Default, Debug)]
struct Config {
    debug: bool,
    port: u16,
    name: String,
}

let config = Config {
    debug: true,
    ..Default::default()
};
```

For HashMap keys, you need both `Eq` and `Hash`:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
struct UserId(u64);
```

## Common Pitfalls

1. **Deriving `Copy` on types with heap data** — If any field is `String`, `Vec`, or another non-Copy type, the derive will fail with a compile error. Use `Clone` alone for these types.
2. **Forgetting `Eq` for HashMap keys** — `PartialEq` alone is not enough. HashMap requires `Eq` (which guarantees reflexivity: `a == a` is always true). Floats implement `PartialEq` but not `Eq`, so they cannot be HashMap keys.

## Best Practices

1. **Always derive `Debug`** — It costs nothing at runtime and makes debugging dramatically easier. There is almost never a reason to omit it.
2. **Derive the minimum set you need** — Do not blindly derive every trait. Each derived trait adds to compile time and creates API commitments. Derive `Clone` when cloning is needed, `Copy` only for small, stack-only types.

## Summary

- `#[derive(...)]` auto-generates trait implementations from struct/enum fields.
- `Debug` should be on virtually every type; `Clone` and `PartialEq` are also common.
- `Copy` requires all fields to be `Copy` and enables implicit duplication.
- `Eq + Hash` are required for HashMap keys.
- `Default` provides zero-value constructors.

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


## Resources

- [Derivable Traits](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html) — Official appendix listing all derivable traits with examples and requirements
- [Derive Macro Reference](https://doc.rust-lang.org/reference/attributes/derive.html) — Rust Reference documentation on the derive attribute and procedural macros

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*