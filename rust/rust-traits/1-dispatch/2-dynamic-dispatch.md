---
source_course: "rust-traits"
source_lesson: "rust-traits-dynamic-dispatch"
---

# Trait Objects: dyn Trait

When you need to mix different types, use trait objects:

```rust
trait Animal {
    fn speak(&self) -> &str;
}

struct Dog;
struct Cat;

impl Animal for Dog {
    fn speak(&self) -> &str { "Woof!" }
}

impl Animal for Cat {
    fn speak(&self) -> &str { "Meow!" }
}

// Heterogeneous collection!
let animals: Vec<Box<dyn Animal>> = vec![
    Box::new(Dog),
    Box::new(Cat),
];

for animal in &animals {
    println!("{}", animal.speak());
}
```

## How It Works: Vtables

```rust
// dyn Trait is a "fat pointer":
// - Pointer to data
// - Pointer to vtable (virtual method table)

// The vtable contains:
// - Drop function
// - Size and alignment
// - Pointers to each trait method
```

## Object Safety Rules

Not all traits can be made into trait objects:

```rust
// Object-safe ✓
trait Drawable {
    fn draw(&self);
}

// NOT object-safe ✗
trait Clone {
    fn clone(&self) -> Self;  // Returns Self
}

trait Sized {  // Sized is not object-safe
    // ...
}

trait Generic {
    fn foo<T>(&self);  // Generic methods
}
```

## Object Safety Requirements

1. No `Self: Sized` bound
2. No associated constants
3. All methods must:
   - Not return `Self`
   - Not have generic type parameters
   - Have `self` receiver (`&self`, `&mut self`, `Box<Self>`, etc.)

See [Trait Objects](https://doc.rust-lang.org/book/ch17-02-trait-objects.html).

## Code Examples

**Trait object forms**

```rust
// Different forms of trait objects
trait Draw {
    fn draw(&self);
}

// Box<dyn Trait> - owned, heap allocated
let shape: Box<dyn Draw> = Box::new(Circle);

// &dyn Trait - borrowed
let shape_ref: &dyn Draw = &Circle;

// Arc<dyn Trait> - shared ownership
let shape_shared: Arc<dyn Draw> = Arc::new(Circle);

// In function parameters
fn render(shapes: &[&dyn Draw]) {
    for shape in shapes {
        shape.draw();
    }
}

// Return trait object
fn create_shape(kind: &str) -> Box<dyn Draw> {
    match kind {
        "circle" => Box::new(Circle),
        "square" => Box::new(Square),
        _ => Box::new(Point),
    }
}
```


## Resources

- [Trait Objects](https://doc.rust-lang.org/book/ch17-02-trait-objects.html) — Using Trait Objects

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*