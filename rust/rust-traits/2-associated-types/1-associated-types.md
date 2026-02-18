---
source_course: "rust-traits"
source_lesson: "rust-traits-associated-types"
---

# Associated Types vs Generic Parameters

## The Problem with Generic Traits

```rust
// Generic parameter version
trait Iterator<Item> {
    fn next(&mut self) -> Option<Item>;
}

// A type could implement this multiple times:
impl Iterator<i32> for MyType { ... }
impl Iterator<String> for MyType { ... }  // Valid!

// But for iterators, this doesn't make sense
// An iterator produces ONE type of item
```

## Associated Types: One Implementation

```rust
trait Iterator {
    type Item;  // Associated type
    fn next(&mut self) -> Option<Self::Item>;
}

impl Iterator for Counter {
    type Item = u32;  // Specify once
    fn next(&mut self) -> Option<u32> { ... }
}
```

## When to Use Each

| Use Associated Types When | Use Generics When |
|---------------------------|-------------------|
| One implementation per type | Multiple implementations needed |
| The type is an "output" | The type is an "input" |
| Part of the trait's identity | Configuration parameter |

## Examples in std

```rust
// Associated types (one implementation)
trait Iterator { type Item; }        // What it yields
trait Deref { type Target; }         // What it derefs to
trait IntoIterator { type Item; type IntoIter; }

// Generic parameters (multiple implementations)
trait From<T> { fn from(t: T) -> Self; }  // Many sources
trait Into<T> { fn into(self) -> T; }     // Many destinations
trait AsRef<T> { fn as_ref(&self) -> &T; }
```

See [Associated Types](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#specifying-placeholder-types-in-trait-definitions-with-associated-types).

## Code Examples

**Multiple associated types**

```rust
// Multiple associated types
trait Graph {
    type Node;
    type Edge;
    
    fn nodes(&self) -> Vec<Self::Node>;
    fn edges(&self) -> Vec<Self::Edge>;
    fn neighbors(&self, node: &Self::Node) -> Vec<Self::Node>;
}

struct CityMap;

impl Graph for CityMap {
    type Node = String;           // City name
    type Edge = (String, String); // Connection
    
    fn nodes(&self) -> Vec<String> { todo!() }
    fn edges(&self) -> Vec<(String, String)> { todo!() }
    fn neighbors(&self, node: &String) -> Vec<String> { todo!() }
}
```


---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*