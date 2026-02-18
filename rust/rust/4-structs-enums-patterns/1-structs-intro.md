---
source_course: "rust"
source_lesson: "rust-structs-intro"
---

# Structs: Custom Data Types

Structs are similar to classes in other languages (without inheritance).

## Named Structs

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

## Field Init Shorthand

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,      // Same as email: email
        username,   // Same as username: username
        active: true,
        sign_in_count: 1,
    }
}
```

## Struct Update Syntax

```rust
let user2 = User {
    email: String::from("another@example.com"),
    ..user1 // Copy remaining fields from user1
};
// Note: user1.username was MOVED to user2!
```

## Tuple Structs

Structs without named fields:

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
// black and origin are different types!
```

## Unit-Like Structs

```rust
struct AlwaysEqual; // No fields
let subject = AlwaysEqual;
```

See [Defining Structs](https://doc.rust-lang.org/stable/book/ch05-01-defining-structs.html).

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


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*