---
source_course: "rust-ownership"
source_lesson: "rust-ownership-box-basics"
---

# Box<T>: Heap Allocation

## Introduction
Box is the simplest smart pointer in Rust — it allocates a value on the heap and provides single ownership semantics. Despite its simplicity, Box is essential for recursive types, large data transfers, and trait objects.

## Key Concepts
- **Box<T>**: A smart pointer that allocates T on the heap with a single owner. When the Box is dropped, the heap memory is freed.
- **Deref coercion**: Box implements `Deref`, so a `&Box<T>` automatically coerces to `&T` in function calls.
- **Recursive type**: A type that contains itself, which has infinite size unless indirected through a pointer like Box.

## Real World Context
You use Box every time you build a tree, linked list, or any recursive data structure. It is also the standard way to create trait objects (`Box<dyn Trait>`) for dynamic dispatch. In performance-sensitive code, Box helps move large data without copying it.

## Deep Dive
Creating a Box allocates on the heap and returns a pointer:

```rust
let b = Box::new(5);
println!("b = {b}"); // Deref coercion: acts like &i32
```

The most common use case is recursive types. Without indirection, the compiler cannot determine the size:

```rust
// Won't compile — infinite size!
// enum List { Cons(i32, List), Nil }

// Works — Box has known pointer size
enum List {
    Cons(i32, Box<List>),
    Nil,
}

let list = List::Cons(1,
    Box::new(List::Cons(2,
        Box::new(List::Nil))));
```

Box also enables moving large data without copying the underlying bytes:

```rust
let huge_array = Box::new([0u8; 1_000_000]);
let moved = huge_array;  // Moves the pointer, not the data!
```

For trait objects, Box provides owned dynamic dispatch:

```rust
trait Animal { fn speak(&self); }
struct Dog;
impl Animal for Dog { fn speak(&self) { println!("Woof"); } }

let animal: Box<dyn Animal> = Box::new(Dog);
animal.speak();  // Dynamic dispatch through vtable
```

## Common Pitfalls
1. **Using Box when a reference suffices** — Box allocates on the heap, which has overhead. If you only need to borrow data temporarily, use a reference instead.
2. **Forgetting deref coercion** — You do not need to manually dereference a Box in most contexts. `&Box<String>` automatically coerces to `&String` to `&str`.

## Best Practices
1. **Use Box for recursive types** — It is the idiomatic solution for cons-lists, trees, and other recursive structures.
2. **Prefer `Box::leak` over `unsafe` for creating `'static` references** — When you need a `'static` reference from owned data, `Box::leak` is the safe and idiomatic approach.

## Summary
- Box<T> is a single-owner heap pointer that frees memory on drop.
- It enables recursive types by providing a fixed-size indirection.
- Deref coercion lets Box behave transparently like a reference.
- Box<dyn Trait> is the standard way to create owned trait objects.

## Code Examples

**Binary tree structure — each Node owns its children through Box, giving the recursive type a known size**

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


## Resources

- [Box<T> Documentation](https://doc.rust-lang.org/std/boxed/struct.Box.html) — Official API reference for Box

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*