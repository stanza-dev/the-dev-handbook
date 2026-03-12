---
source_course: "rust"
source_lesson: "rust-methods-impl"
---

# Methods and Associated Functions

## Introduction

Structs on their own are just data containers. The `impl` block is where you add behavior to your types by defining methods and associated functions. Methods operate on instances of a struct, while associated functions act as constructors or utility functions. Together, they provide encapsulation similar to what classes offer in other languages.

## Key Concepts

- **Method**: A function defined inside an `impl` block that takes `self` (or a reference to it) as its first parameter. Called with dot syntax: `instance.method()`.
- **Associated function**: A function inside an `impl` block that does NOT take `self`. Called with `::` syntax: `Type::function()`. Often used as constructors.
- **Self receiver**: The first parameter of a method. `&self` borrows immutably, `&mut self` borrows mutably, and `self` takes ownership.

## Real World Context

Every real Rust project defines methods on its structs. A `Database` struct has a `.query()` method. An `HttpRequest` struct has `.headers()` and `.body()` methods. The `impl` block is how you associate logic with data, keeping your codebase organized and your APIs intuitive.

## Deep Dive

Define methods inside an `impl` block. The method receiver determines what level of access the method has:

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Immutable borrow — reads data only
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // Mutable borrow — can modify the instance
    fn double(&mut self) {
        self.width *= 2;
        self.height *= 2;
    }

    // Takes ownership — consumes the instance
    fn into_square(self) -> Rectangle {
        let side = self.width.max(self.height);
        Rectangle { width: side, height: side }
    }
}
```

The three receiver types map directly to ownership and borrowing rules:

| Receiver | Access | Ownership |
|----------|--------|-----------|
| `&self` | Read only | Borrowed |
| `&mut self` | Read/write | Mutably borrowed |
| `self` | Full | Owned (consumed) |

Associated functions do not take `self` and are often used as constructors:

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self { width: size, height: size }
    }
}

let sq = Rectangle::square(10);
```

Note the use of `Self` as a shorthand for the implementing type.

## Common Pitfalls

1. **Using `self` when `&self` suffices** — Taking `self` by value consumes the instance, making it unusable afterward. Most methods should use `&self` or `&mut self` unless the method is intentionally transforming the value.
2. **Confusing method calls and associated function calls** — Methods use dot syntax (`rect.area()`), while associated functions use `::` syntax (`Rectangle::square(10)`). Mixing them up causes compiler errors.

## Best Practices

1. **Implement a `new` constructor** — By convention, Rust types provide a `fn new(...) -> Self` associated function. This is the standard constructor pattern in the ecosystem.
2. **Use `Self` instead of repeating the type name** — Inside an `impl` block, `Self` refers to the type being implemented. It makes refactoring easier and code more readable.

## Summary

- Methods are defined in `impl` blocks and take `&self`, `&mut self`, or `self` as the first parameter.
- The receiver type controls whether the method reads, modifies, or consumes the instance.
- Associated functions have no `self` parameter and are called with `::` syntax; they are often constructors.
- Use `Self` as a shorthand for the implementing type inside `impl` blocks.
- Follow the `new` constructor convention for idiomatic Rust APIs.

## Code Examples

**Constructor pattern**

```rust
impl Rectangle {
    fn new(width: u32, height: u32) -> Self {
        Self { width, height } // Self = Rectangle
    }
    
    fn is_square(&self) -> bool {
        self.width == self.height
    }
}

let rect = Rectangle::new(10, 20);
println!("Is square? {}", rect.is_square());
```


## Resources

- [Method Syntax](https://doc.rust-lang.org/stable/book/ch05-03-method-syntax.html) — Official guide to defining methods on structs
- [Rust By Example - Methods](https://doc.rust-lang.org/rust-by-example/fn/methods.html) — Hands-on examples of methods and associated functions

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*