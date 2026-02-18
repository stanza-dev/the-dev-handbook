---
source_course: "rust-traits"
source_lesson: "rust-traits-gats"
---

# GATs: Associated Types with Generics

Stabilized in Rust 1.65, GATs let associated types have their own generic parameters.

## The Problem: Streaming Iterator

```rust
// Standard Iterator can't return references to self
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

// We want:
impl Iterator for WindowsIterator {
    type Item = &[u8];  // Error! Lifetime needed
    // But Item doesn't have a lifetime parameter!
}
```

## GATs to the Rescue

```rust
trait LendingIterator {
    type Item<'a> where Self: 'a;  // GAT!
    
    fn next(&mut self) -> Option<Self::Item<'_>>;
}

impl LendingIterator for WindowsIterator {
    type Item<'a> = &'a [u8] where Self: 'a;
    
    fn next(&mut self) -> Option<&[u8]> {
        // Can return reference tied to self's lifetime!
    }
}
```

## GAT Examples

```rust
// Generic over type parameter
trait Container {
    type Item<T>;
    
    fn get<T>(&self) -> Self::Item<T>;
}

// Generic over lifetime AND type
trait Parse {
    type Output<'a, T> where T: 'a;
    
    fn parse<'a, T>(&'a self) -> Self::Output<'a, T>;
}
```

## Real-World Use Case: Async Traits

```rust
// Before GATs, async trait methods were hard
trait AsyncIterator {
    type Item;
    type Future<'a>: Future<Output = Option<Self::Item>> 
        where Self: 'a;
    
    fn next(&mut self) -> Self::Future<'_>;
}
```

See [GATs](https://blog.rust-lang.org/2022/10/28/gats-stabilization.html).

## Code Examples

**GATs for connection pools**

```rust
// Database connection pool pattern with GATs
trait Pool {
    type Connection<'pool> where Self: 'pool;
    
    fn get(&self) -> Self::Connection<'_>;
}

struct PostgresPool { /* ... */ }
struct PostgresConn<'a> { pool: &'a PostgresPool }

impl Pool for PostgresPool {
    type Connection<'pool> = PostgresConn<'pool> where Self: 'pool;
    
    fn get(&self) -> PostgresConn<'_> {
        PostgresConn { pool: self }
    }
}

// The connection can reference the pool!
fn use_pool(pool: &PostgresPool) {
    let conn = pool.get();  // Borrows pool
    // conn.execute(...);
}   // conn dropped, borrow ends
```


## Resources

- [GATs Stabilization](https://blog.rust-lang.org/2022/10/28/gats-stabilization.html) — GATs announcement blog post

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*