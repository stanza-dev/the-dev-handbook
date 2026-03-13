---
source_course: "rust-traits"
source_lesson: "rust-traits-type-families"
---

# Type Families & Associated Type Patterns

## Introduction
Associated types can model "type families" — groups of related types that vary together. This pattern appears in collection traits, driver abstractions, and platform-specific APIs where multiple associated types must be consistent with each other.

## Key Concepts
- **Type Family**: A set of associated types in a trait that are semantically related and vary together.
- **Coherent Abstraction**: When associated types constrain each other to ensure type-level consistency.
- **Associated Type Defaults**: Default types in traits that implementors may override (unstable for some patterns, but simple defaults work since Rust 1.41).

## Real World Context
The `std::io::Read` and `std::io::Write` traits use associated error types. Database abstraction layers define type families for connections, transactions, and query results. Platform abstraction crates use type families so that `Window`, `EventLoop`, and `Renderer` types all match the target platform.

## Deep Dive

### Collection Type Family

```rust
trait Collection {
    type Item;
    type Iter<'a>: Iterator<Item = &'a Self::Item> where Self: 'a;

    fn add(&mut self, item: Self::Item);
    fn iter(&self) -> Self::Iter<'_>;
    fn len(&self) -> usize;
}

impl Collection for Vec<String> {
    type Item = String;
    type Iter<'a> = std::slice::Iter<'a, String>;

    fn add(&mut self, item: String) { self.push(item); }
    fn iter(&self) -> std::slice::Iter<'_, String> { self.as_slice().iter() }
    fn len(&self) -> usize { self.len() }
}
```

The `Item` and `Iter` types are linked — `Iter` yields references to `Item`. This relationship is enforced at the type level.

### Platform Abstraction Pattern

```rust
trait Platform {
    type Window;
    type Renderer;
    type Event;

    fn create_window(&self) -> Self::Window;
    fn create_renderer(&self, window: &Self::Window) -> Self::Renderer;
    fn poll_event(&self) -> Option<Self::Event>;
}
```

All three types are determined by the single choice of `Platform` implementation, ensuring they are always compatible.

### Constraining Associated Types Against Each Other

```rust
trait Codec {
    type Encoded: AsRef<[u8]>;
    type Error: std::error::Error;

    fn encode(&self, data: &[u8]) -> Result<Self::Encoded, Self::Error>;
    fn decode(&self, data: &Self::Encoded) -> Result<Vec<u8>, Self::Error>;
}
```

The `Encoded` type carries a bound (`AsRef<[u8]>`), and the same `Error` type is used for both operations.

## Common Pitfalls
1. **Too many associated types** — A trait with 10 associated types is hard to implement. Consider splitting into smaller traits.
2. **Missing bounds on associated types** — If generic code needs to print an associated type, add `type Item: Display` in the trait.

## Best Practices
1. **Group related types into a single trait** — If types must be consistent, put them in one trait so a single impl determines all of them.
2. **Add bounds to associated types** — Require `Display`, `Debug`, `Send`, etc. on associated types when the trait's methods need those capabilities.

## Summary
- Type families group related associated types that must be consistent.
- One trait implementation determines all associated types at once.
- Bounds on associated types (`type Item: Display`) ensure usability in generic contexts.
- Split large type families into smaller, composable traits.

## Code Examples

**A database trait where Connection, Query, and Row types form a type family — choosing Postgres determines all three types together**

```rust
// Type family pattern: Database abstraction
trait Database {
    type Connection;
    type Query;
    type Row;

    fn connect(&self, url: &str) -> Self::Connection;
    fn query(&self, conn: &Self::Connection, sql: &str) -> Self::Query;
    fn fetch(&self, query: Self::Query) -> Vec<Self::Row>;
}

// One impl determines all three types
struct Postgres;
impl Database for Postgres {
    type Connection = PgConnection;
    type Query = PgQuery;
    type Row = PgRow;

    fn connect(&self, _url: &str) -> PgConnection { todo!() }
    fn query(&self, _conn: &PgConnection, _sql: &str) -> PgQuery { todo!() }
    fn fetch(&self, _query: PgQuery) -> Vec<PgRow> { todo!() }
}

struct PgConnection;
struct PgQuery;
struct PgRow;
```


## Resources

- [Associated Types](https://doc.rust-lang.org/book/ch20-03-advanced-traits.html) — Rust Book on associated types and their advanced patterns

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*