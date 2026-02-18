---
source_course: "rust-ownership"
source_lesson: "rust-ownership-box-basics"
---

# Box<T>: Single-Owner Heap Pointer

Box is the simplest smart pointer - it allocates data on the heap with single ownership.

```rust
let b = Box::new(5);
println!("b = {b}"); // Deref coercion: acts like &i32
```

## When to Use Box

### 1. Recursive Types

```rust
// Won't compile - infinite size!
// enum List { Cons(i32, List), Nil }

// Works - Box has known size
enum List {
    Cons(i32, Box<List>),
    Nil,
}

let list = List::Cons(1, 
    Box::new(List::Cons(2, 
        Box::new(List::Nil))));
```

### 2. Large Data Transfer

```rust
// Move large data without copying
let huge_array = Box::new([0u8; 1_000_000]);
let moved = huge_array;  // Moves pointer, not data!
```

### 3. Trait Objects

```rust
trait Animal { fn speak(&self); }

let animal: Box<dyn Animal> = Box::new(Dog);
animal.speak();  // Dynamic dispatch
```

## Deref Coercion

Box implements `Deref`, so it auto-dereferences:

```rust
fn takes_ref(s: &str) { println!("{s}"); }

let boxed = Box::new(String::from("hello"));
takes_ref(&boxed);  // Auto: &Box<String> → &String → &str
```

## Box::leak for 'static

```rust
let leaked: &'static str = Box::leak(String::from("permanent").into_boxed_str());
```

See [Box<T>](https://doc.rust-lang.org/std/boxed/struct.Box.html).

## Code Examples

**Binary tree structure**

```rust
// Binary tree with Box
#[derive(Debug)]
enum Tree<T> {
    Leaf(T),
    Node {
        left: Box<Tree<T>>,
        right: Box<Tree<T>>,
    },
}

let tree = Tree::Node {
    left: Box::new(Tree::Leaf(1)),
    right: Box::new(Tree::Node {
        left: Box::new(Tree::Leaf(2)),
        right: Box::new(Tree::Leaf(3)),
    }),
};
```


---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*