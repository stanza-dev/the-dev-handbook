---
source_course: "rust-functional"
source_lesson: "rust-func-option-combinators"
---

# Option<T> Combinators

Avoid match boilerplate with combinators.

## Transforming Values

```rust
let x: Option<i32> = Some(5);

// map: Transform the inner value
let doubled = x.map(|n| n * 2);  // Some(10)

// map_or: Transform or provide default
let doubled = x.map_or(0, |n| n * 2);  // 10

// map_or_else: Lazy default
let doubled = x.map_or_else(|| expensive_default(), |n| n * 2);
```

## Chaining Options

```rust
fn get_user(id: u32) -> Option<User> { ... }
fn get_email(user: &User) -> Option<String> { ... }

// and_then: Chain operations that return Option
let email = get_user(1)
    .and_then(|user| get_email(&user));

// Equivalent to:
let email = match get_user(1) {
    Some(user) => get_email(&user),
    None => None,
};
```

## Providing Defaults

```rust
let x: Option<i32> = None;

// unwrap_or: Provide default value
let value = x.unwrap_or(42);  // 42

// unwrap_or_else: Lazy default
let value = x.unwrap_or_else(|| compute_default());

// unwrap_or_default: Use Default trait
let value = x.unwrap_or_default();  // 0 for i32

// or: Provide alternative Option
let value = x.or(Some(42));  // Some(42)
```

## Filtering

```rust
let x = Some(4);

// filter: Keep Some if predicate passes
let even = x.filter(|&n| n % 2 == 0);  // Some(4)
let odd = x.filter(|&n| n % 2 != 0);   // None
```

## Flattening

```rust
let x: Option<Option<i32>> = Some(Some(5));
let flat = x.flatten();  // Some(5)
```

## Code Examples

**Option combinator patterns**

```rust
// Real-world combinator chain
struct User {
    name: String,
    email: Option<String>,
    age: Option<u32>,
}

fn process_user(user: Option<User>) -> String {
    user
        .filter(|u| u.age.map_or(false, |a| a >= 18))
        .and_then(|u| u.email)
        .map(|email| format!("Contact: {email}"))
        .unwrap_or_else(|| "No valid contact".to_string())
}

// Combining with ?
fn get_config_value(config: &Option<Config>) -> Option<String> {
    config.as_ref()?.settings.get("key")?.value.clone()
}

// Converting to Result
let x: Option<i32> = Some(5);
let result: Result<i32, &str> = x.ok_or("Value was None");

// From Result to Option
let r: Result<i32, &str> = Ok(5);
let opt: Option<i32> = r.ok();  // Some(5)
```


---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*