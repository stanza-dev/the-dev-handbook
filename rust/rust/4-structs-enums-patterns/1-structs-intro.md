---
source_course: "rust"
source_lesson: "rust-structs-intro"
---

# Defining and Using Structs

## Introduction

Structs are Rust's primary mechanism for creating custom data types. If you come from object-oriented languages, think of structs as classes without inheritance. They group related data together and form the foundation of how you model your domain in Rust.

## Key Concepts

- **Named struct**: A struct with named fields, the most common form. Each field has a name and a type.
- **Tuple struct**: A struct with unnamed, positionally accessed fields. Useful for lightweight wrappers like `Color(u8, u8, u8)`.
- **Unit-like struct**: A struct with no fields at all, used as a marker type or for trait implementations.
- **Struct update syntax (`..`)**: A shorthand for creating a new struct instance that copies most fields from an existing one.

## Real World Context

Every Rust application uses structs to model its domain. A web server defines structs for requests and responses. A game engine defines structs for entities, positions, and velocities. Structs combined with `impl` blocks give you encapsulation and behavior without the complexity of class hierarchies.

## Deep Dive

Define a named struct and create an instance by providing values for every field:

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

let user1 = User {
    email: String::from("alice@example.com"),
    username: String::from("alice123"),
    active: true,
    sign_in_count: 1,
};
```

The field init shorthand lets you omit the field name when the variable has the same name:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,       // same as email: email
        username,    // same as username: username
        active: true,
        sign_in_count: 1,
    }
}
```

Struct update syntax creates a new instance from an existing one, overriding specific fields:

```rust
let user2 = User {
    email: String::from("bob@example.com"),
    ..user1 // remaining fields come from user1
};
// Note: user1.username was MOVED to user2
```

Tuple structs are useful when you want a distinct type without named fields:

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
// black and origin are different types!
```

## Common Pitfalls

1. **Forgetting that struct update syntax moves String fields** — When you use `..user1`, any `String` or other non-Copy fields are moved from `user1`. After the update, `user1` may be partially invalid. If you need to keep the original, clone the fields explicitly.
2. **Not deriving Debug** — Structs cannot be printed with `{:?}` by default. Add `#[derive(Debug)]` to enable debug formatting during development.

## Best Practices

1. **Derive common traits on your structs** — Add `#[derive(Debug, Clone, PartialEq)]` from the start. It costs nothing at compile time and saves you from adding them later when you need equality checks or debugging.
2. **Use tuple structs for newtype patterns** — Wrapping a primitive in a tuple struct (e.g., `struct UserId(u64)`) gives you type safety and prevents mixing up semantically different values that share the same underlying type.

## Summary

- Named structs group related data with named fields.
- Field init shorthand and struct update syntax reduce boilerplate when creating instances.
- Tuple structs provide distinct types without named fields, ideal for the newtype pattern.
- Struct update syntax moves non-Copy fields from the source, which may invalidate it.
- Always derive `Debug` on your structs during development.

## Code Examples

**Debug printing structs**

```rust
#[derive(Debug)] // Allows {:?} formatting
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect = Rectangle { width: 30, height: 50 };
    println!("rect is {:?}", rect);
    println!("rect is {:#?}", rect); // Pretty print
}
```


## Resources

- [Defining Structs](https://doc.rust-lang.org/stable/book/ch05-01-defining-structs.html) — Official guide to defining and instantiating structs
- [Rust By Example - Structs](https://doc.rust-lang.org/rust-by-example/custom_types/structs.html) — Hands-on struct examples in Rust By Example

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*