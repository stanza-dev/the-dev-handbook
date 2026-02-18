---
source_course: "rust"
source_lesson: "rust-generics-basics"
---

# Generics

Generics allow us to write code that works on multiple types without duplication.

## In Functions

```rust
// Without generics: duplicate code
fn largest_i32(list: &[i32]) -> &i32 { /* ... */ }
fn largest_char(list: &[char]) -> &char { /* ... */ }

// With generics: single function
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}
```

## In Structs

```rust
struct Point<T> {
    x: T,
    y: T,
}

let integer = Point { x: 5, y: 10 };
let float = Point { x: 1.0, y: 4.0 };

// Multiple type parameters
struct Point2<T, U> {
    x: T,
    y: U,
}

let mixed = Point2 { x: 5, y: 4.0 };
```

## In Methods

```rust
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

// Methods only for specific types
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

## Zero-Cost Abstraction

Rust implements generics using **monomorphization**: the compiler generates specific code for each concrete type used.

```rust
let integer = Some(5);  // Compiler generates Option_i32
let float = Some(5.0);  // Compiler generates Option_f64
// No runtime cost!
```

See [Generic Data Types](https://doc.rust-lang.org/stable/book/ch10-01-syntax.html).

## Code Examples

**Multiple trait bounds**

```rust
// Generic function with multiple bounds
fn print_debug<T: std::fmt::Debug + Clone>(item: T) {
    let cloned = item.clone();
    println!("{:?}", cloned);
}

// Using where clause for readability
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: std::fmt::Display + Clone,
    U: Clone + std::fmt::Debug,
{
    // ...
    0
}
```


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*