---
source_course: "rust-traits"
source_lesson: "rust-traits-gats"
---

# Generic Associated Types (GATs)

## Introduction
Stabilized in Rust 1.65, GATs let associated types have their own generic parameters (lifetimes or types). This unlocks patterns like lending iterators, where an iterator can return references tied to its own lifetime.

## Key Concepts
- **GAT**: An associated type parameterized by lifetimes or types: `type Item<'a> where Self: 'a;`
- **Lending Iterator**: An iterator that can yield references borrowing from itself, impossible with the standard `Iterator` trait.
- **Self-referential Borrows**: GATs enable associated types whose lifetime is tied to `&self`.

## Real World Context
Database connection pools, streaming parsers, and zero-copy deserialization all benefit from GATs. A pool can return a connection whose lifetime is tied to the pool, enforced at the type level.

## Deep Dive
The standard `Iterator` trait cannot return references tied to itself:

```rust
trait Iterator {
    type Item; // No lifetime parameter!
    fn next(&mut self) -> Option<Self::Item>;
}
// Cannot set type Item = &'??? [u8] — no lifetime to use!
```

GATs solve this by parameterizing the associated type:

```rust
trait LendingIterator {
    type Item<'a> where Self: 'a; // GAT with lifetime

    fn next(&mut self) -> Option<Self::Item<'_>>;
}

impl LendingIterator for WindowsIter {
    type Item<'a> = &'a [u8] where Self: 'a;

    fn next(&mut self) -> Option<&[u8]> {
        // Can return a reference tied to self's lifetime
        todo!()
    }
}
```

GATs can also be parameterized by types:

```rust
trait Container {
    type Item<T>;
    fn wrap<T>(&self, value: T) -> Self::Item<T>;
}
```

### Connection Pool Pattern

```rust
trait Pool {
    type Connection<'pool> where Self: 'pool;
    fn get(&self) -> Self::Connection<'_>;
}

impl Pool for PostgresPool {
    type Connection<'pool> = PgConn<'pool> where Self: 'pool;
    fn get(&self) -> PgConn<'_> { PgConn { pool: self } }
}
```

The connection borrows the pool, and the compiler enforces the lifetime relationship.

## Common Pitfalls
1. **Forgetting `where Self: 'a`** — GATs with lifetime parameters almost always need this bound to ensure the implementing type outlives the borrow.
2. **Overusing GATs** — Standard associated types suffice when you do not need lifetime or type parameterization.

## Best Practices
1. **Use GATs for self-referential borrows** — When an associated type needs to borrow from `&self`, a GAT with a lifetime parameter is the right tool.
2. **Prefer standard associated types when possible** — GATs add complexity; only reach for them when the simpler form cannot express what you need.

## Summary
- GATs are associated types with their own generic parameters.
- They enable lending iterators and self-referential borrows.
- The `where Self: 'a` bound is almost always required.
- Stabilized in Rust 1.65.

## Code Examples

**A connection pool trait using GATs to tie the connection's lifetime to the pool, ensuring the connection cannot outlive the pool**

```rust
// Database pool with GATs — the connection borrows the pool
trait Pool {
    type Connection<'pool> where Self: 'pool;
    fn get(&self) -> Self::Connection<'_>;
}

struct PostgresPool { /* ... */ }
struct PgConn<'a> { pool: &'a PostgresPool }

impl Pool for PostgresPool {
    type Connection<'pool> = PgConn<'pool> where Self: 'pool;
    fn get(&self) -> PgConn<'_> {
        PgConn { pool: self }
    }
}

fn use_pool(pool: &PostgresPool) {
    let conn = pool.get(); // borrows pool
    // conn.execute(...);
} // conn dropped, borrow released
```


## Resources

- [GATs Stabilization](https://blog.rust-lang.org/2022/10/28/gats-stabilization.html) — Official blog post announcing GAT stabilization with examples

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*