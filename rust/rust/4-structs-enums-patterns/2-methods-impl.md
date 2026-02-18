---
source_course: "rust"
source_lesson: "rust-methods-impl"
---

# Methods: Functions on Structs

Methods are defined inside an `impl` block.

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Method: takes &self as first parameter
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    // Method that mutates
    fn double(&mut self) {
        self.width *= 2;
        self.height *= 2;
    }
    
    // Method that consumes self
    fn into_square(self) -> Rectangle {
        let side = self.width.max(self.height);
        Rectangle { width: side, height: side }
    }
}
```

## Method Receivers

| Receiver | Access | Ownership |
|----------|--------|----------|
| `&self` | Read only | Borrowed |
| `&mut self` | Read/write | Mutably borrowed |
| `self` | Full | Owned (consumed) |

## Associated Functions

Functions without `self` (like static methods):

```rust
impl Rectangle {
    // Associated function (not a method)
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}

let sq = Rectangle::square(10); // Call with ::
```

## Multiple impl Blocks

```rust
impl Rectangle {
    fn area(&self) -> u32 { self.width * self.height }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

See [Method Syntax](https://doc.rust-lang.org/stable/book/ch05-03-method-syntax.html).

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


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*