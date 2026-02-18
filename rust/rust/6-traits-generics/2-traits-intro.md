---
source_course: "rust"
source_lesson: "rust-traits-intro"
---

# Traits

A trait tells the Rust compiler about functionality a particular type has. It is similar to interfaces in other languages.

## Defining a Trait

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

## Implementing a Trait

```rust
pub struct NewsArticle {
    pub headline: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {}", self.headline, self.author)
    }
}
```

## Default Implementations

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;
    
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

## Traits as Parameters

```rust
// impl Trait syntax
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// Trait bound syntax (equivalent)
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

// Multiple bounds
pub fn notify(item: &(impl Summary + std::fmt::Display)) { }
```

## Returning Traits

```rust
fn returns_summarizable() -> impl Summary {
    NewsArticle {
        headline: String::from("Penguins win!"),
        author: String::from("Iceburgh"),
        content: String::from("The penguins won..."),
    }
}
```

See [Traits: Defining Shared Behavior](https://doc.rust-lang.org/stable/book/ch10-02-traits.html).

## Code Examples

**Blanket implementations**

```rust
use std::fmt::Display;

// Conditional implementation
impl<T: Display> ToString for T {
    // Any type that implements Display
    // automatically gets ToString!
}

// Blanket implementations in std
let s = 3.to_string(); // Works because i32: Display
```


## Resources

- [Traits](https://doc.rust-lang.org/stable/book/ch10-02-traits.html) — Complete guide to traits

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*