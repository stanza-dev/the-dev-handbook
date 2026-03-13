---
source_course: "rust-traits"
source_lesson: "rust-traits-associated-types"
---

# Associated Types

## Introduction
Associated types let a trait declare placeholder types that implementors fill in. Unlike generic parameters, associated types enforce that each implementation chooses exactly one type, making them ideal for "output" types.

## Key Concepts
- **Associated Type**: A type placeholder declared inside a trait with `type Name;`.
- **Output Type**: An associated type that the trait produces or works with, determined by the implementor.
- **Generic Parameter**: A type the caller chooses (input), as opposed to associated types which the implementor chooses (output).

## Real World Context
`Iterator::Item`, `Deref::Target`, and `Add::Output` are all associated types. They enforce a single canonical choice per implementation, preventing confusion about which type an iterator yields.

## Deep Dive
With generic parameters, a type could implement a trait multiple times:

```rust
// If Iterator used a generic parameter:
trait Iterator<Item> {
    fn next(&mut self) -> Option<Item>;
}
// A type could implement Iterator<i32> AND Iterator<String>!
```

Associated types fix this by allowing only one implementation per type:

```rust
trait Iterator {
    type Item; // Associated type
    fn next(&mut self) -> Option<Self::Item>;
}

impl Iterator for Counter {
    type Item = u32; // One choice, made by the implementor
    fn next(&mut self) -> Option<u32> { /* ... */ }
}
```

### When to Use Each

| Associated Types | Generic Parameters |
|-----------------|-------------------|
| One implementation per type | Multiple implementations needed |
| Type is an "output" | Type is an "input" |
| `Iterator { type Item; }` | `From<T>`, `Into<T>`, `AsRef<T>` |

### Associated Type Bounds

You can constrain associated types in where clauses and trait bounds:

```rust
fn sum_iter<I: Iterator<Item = i32>>(iter: I) -> i32 {
    iter.fold(0, |a, b| a + b)
}

fn print_all<I>(iter: I)
where
    I: Iterator,
    I::Item: Display,
{
    for item in iter { println!("{item}"); }
}
```

## Common Pitfalls
1. **Using generics when associated types are appropriate** — If a type should implement the trait only once, use an associated type. Using generics would allow conflicting implementations.
2. **Forgetting to bind associated types in generic code** — When writing generic functions, you often need `where I::Item: SomeTrait` to use methods on the associated type.

## Best Practices
1. **Use associated types for output/result types** — If the type is determined by the implementor (not the caller), it should be an associated type.
2. **Use generics for input/configuration types** — If the caller should choose the type, use a generic parameter (`From<T>`, `AsRef<T>`).

## Summary
- Associated types declare placeholder types that each implementor fills in exactly once.
- They prevent multiple conflicting implementations of the same trait.
- Use associated types for outputs, generics for inputs.
- Constrain associated types with `Iterator<Item = T>` or `where I::Item: Trait`.

## Code Examples

**A Graph trait with two associated types (Node and Edge) — each implementor defines exactly what types represent nodes and edges**

```rust
// Multiple associated types in a Graph trait
trait Graph {
    type Node;
    type Edge;

    fn nodes(&self) -> Vec<Self::Node>;
    fn edges(&self) -> Vec<Self::Edge>;
    fn neighbors(&self, node: &Self::Node) -> Vec<Self::Node>;
}

struct CityMap;

impl Graph for CityMap {
    type Node = String;            // City name
    type Edge = (String, String);  // Connection

    fn nodes(&self) -> Vec<String> { todo!() }
    fn edges(&self) -> Vec<(String, String)> { todo!() }
    fn neighbors(&self, _node: &String) -> Vec<String> { todo!() }
}
```


## Resources

- [Associated Types](https://doc.rust-lang.org/book/ch20-03-advanced-traits.html) — Official Rust Book chapter on associated types in trait definitions

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*