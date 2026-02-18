---
source_course: "rust-ownership"
source_lesson: "rust-ownership-cell-basics"
---

# Cell<T>: Mutate Through Shared References

`Cell<T>` allows mutation through a shared reference for `Copy` types.

```rust
use std::cell::Cell;

let x = Cell::new(5);
let y = &x;
let z = &x;

y.set(10);  // Mutate through shared ref!
z.set(20);

println!("{}", x.get());  // 20
```

## How Cell Works

`Cell` doesn't give you a reference to the inner value. Instead:
- `get()` - Returns a **copy** of the value
- `set()` - Replaces the value
- `replace()` - Swaps and returns old value

```rust
let c = Cell::new(5);
let old = c.replace(10);  // old = 5, cell now 10
```

## Limitations

1. **Only works for `Copy` types** (or use `take()`)
2. **Not thread-safe** - `Cell` is `!Sync`
3. **No borrowing the inner value**

## Use Cases

### Counters in shared structs:
```rust
struct Stats {
    hits: Cell<usize>,
}

impl Stats {
    fn record_hit(&self) {
        self.hits.set(self.hits.get() + 1);
    }
}
```

### Caching computed values:
```rust
struct Computed {
    value: i32,
    cached_square: Cell<Option<i32>>,
}

impl Computed {
    fn square(&self) -> i32 {
        self.cached_square.get().unwrap_or_else(|| {
            let sq = self.value * self.value;
            self.cached_square.set(Some(sq));
            sq
        })
    }
}
```

See [Cell](https://doc.rust-lang.org/std/cell/struct.Cell.html).

## Code Examples

**Counter with Cell**

```rust
use std::cell::Cell;

// Interior mutability in immutable struct
struct Counter {
    count: Cell<usize>,
}

impl Counter {
    fn new() -> Self {
        Counter { count: Cell::new(0) }
    }
    
    fn increment(&self) {  // Note: &self, not &mut self
        self.count.set(self.count.get() + 1);
    }
    
    fn get(&self) -> usize {
        self.count.get()
    }
}

let counter = Counter::new();
counter.increment();
counter.increment();
assert_eq!(counter.get(), 2);
```


---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*