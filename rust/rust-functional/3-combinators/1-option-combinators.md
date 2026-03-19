---
source_course: "rust-functional"
source_lesson: "rust-func-option-combinators"
---

# Option Combinators

## Introduction
`Option<T>` is Rust's way of representing a value that may or may not exist, replacing null pointers with a safe, composable type. Rather than writing nested `match` expressions, you can chain combinator methods to transform, filter, and provide defaults for optional values in a functional style.

## Key Concepts
- **`map`**: Transforms the inner value of `Some(T)` into `Some(U)` using a function. Returns `None` if the Option is `None`.
- **`and_then`** (also called flat_map): Like `map`, but the function itself returns an `Option`. Avoids nested `Option<Option<T>>`.
- **`unwrap_or` / `unwrap_or_else`**: Extracts the inner value or provides a default. The `_else` variant computes the default lazily.
- **`filter`**: Converts `Some(T)` to `None` if the value does not satisfy a predicate.

## Real World Context
Option combinators are used everywhere in Rust codebases. Parsing configuration files, looking up database records, accessing nested JSON fields — any operation that might yield "nothing" returns an `Option`. Chaining combinators keeps your code flat and readable, avoiding deeply nested match arms.

## Deep Dive

### Transforming with map

`map` applies a function to the inner value without unwrapping:

```rust
let port: Option<u16> = Some(8080);

let port_str: Option<String> = port.map(|p| format!(":{p}"));
assert_eq!(port_str, Some(":8080".to_string()));

let missing: Option<u16> = None;
let missing_str: Option<String> = missing.map(|p| format!(":{p}"));
assert_eq!(missing_str, None); // map does nothing on None
```

`map` is safe — it never panics and propagates `None` automatically.

### Chaining with and_then

When your transformation itself returns an `Option`, use `and_then` to avoid `Option<Option<T>>`:

```rust
fn find_user(id: u32) -> Option<User> { /* ... */ }
fn get_email(user: &User) -> Option<String> { /* ... */ }

// and_then flattens Option<Option<String>> into Option<String>
let email: Option<String> = find_user(42)
    .and_then(|user| get_email(&user));
```

If `find_user` returns `None`, the entire chain short-circuits to `None` without calling `get_email`.

### Providing defaults

Extract a value or fall back to a default:

```rust
let config_port: Option<u16> = None;

// unwrap_or: eager default
let port = config_port.unwrap_or(3000);
assert_eq!(port, 3000);

// unwrap_or_else: lazy default (only computed when None)
let port = config_port.unwrap_or_else(|| {
    read_default_port_from_env() // Only called if None
});

// unwrap_or_default: uses the Default trait
let port: u16 = config_port.unwrap_or_default(); // 0 for u16
```

Prefer `unwrap_or_else` when the default is expensive to compute.

### Filtering optional values

`filter` converts `Some` to `None` if the predicate fails:

```rust
let age: Option<u32> = Some(15);

let adult_age = age.filter(|&a| a >= 18);
assert_eq!(adult_age, None); // 15 is not >= 18

let valid_age: Option<u32> = Some(25);
let adult = valid_age.filter(|&a| a >= 18);
assert_eq!(adult, Some(25));
```

### Combining into a pipeline

Chain multiple combinators to build expressive pipelines:

```rust
fn process_username(input: Option<&str>) -> String {
    input
        .map(|s| s.trim())
        .filter(|s| !s.is_empty())
        .map(|s| s.to_lowercase())
        .unwrap_or_else(|| "anonymous".to_string())
}

assert_eq!(process_username(Some("  ALICE  ")), "alice");
assert_eq!(process_username(Some("")), "anonymous");
assert_eq!(process_username(None), "anonymous");
```

## Common Pitfalls
1. **Using `unwrap()` in production code** — `unwrap()` panics on `None`. Use `unwrap_or`, `unwrap_or_else`, or propagate with `?` instead.
2. **Using `map` when `and_then` is needed** — If your closure returns `Option<T>`, `map` produces `Option<Option<T>>`. Use `and_then` to flatten.
3. **Ignoring `filter`** — Many developers write `map` + match to conditionally discard values. `filter` does this in one step.

## Best Practices
1. **Chain instead of nesting** — Replace nested `match` or `if let` with a combinator chain for flat, readable code.
2. **Prefer `unwrap_or_else` over `unwrap_or` for expensive defaults** — The closure in `unwrap_or_else` is only invoked when the Option is None.
3. **Use `ok_or` to convert Option to Result** — When absence is an error, `option.ok_or(MyError::NotFound)?` converts cleanly.

## Summary
- `map` transforms the inner value; `None` propagates automatically.
- `and_then` chains operations that themselves return `Option`.
- `unwrap_or` / `unwrap_or_else` provide defaults when the Option is None.
- `filter` conditionally converts Some to None.
- Combinators keep code flat and avoid nested match expressions.

## Code Examples

**Practical Option combinator patterns — filtering users, converting to Result with ok_or, and flattening nested Options**

```rust
// Real-world combinator chain: processing optional user data
struct User {
    name: String,
    email: Option<String>,
    age: Option<u32>,
}

fn get_contact_info(user: Option<User>) -> String {
    user
        .filter(|u| u.age.map_or(false, |a| a >= 18))
        .and_then(|u| u.email)
        .map(|email| format!("Contact: {email}"))
        .unwrap_or_else(|| "No valid contact available".to_string())
}

// Converting between Option and Result
fn find_config_value(key: &str) -> Result<String, String> {
    let config: Option<String> = lookup(key);
    config.ok_or_else(|| format!("Missing config key: {key}"))
}

// Flattening nested Options
let nested: Option<Option<i32>> = Some(Some(42));
let flat: Option<i32> = nested.flatten();
assert_eq!(flat, Some(42));
```


## Resources

- [Option enum documentation](https://doc.rust-lang.org/std/option/enum.Option.html) — Complete API reference for Option with all combinator methods and examples

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*