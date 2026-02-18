---
source_course: "rust-traits"
source_lesson: "rust-traits-impl-trait"
---

# impl Trait

`impl Trait` lets you hide the concrete type while keeping static dispatch.

## In Return Position

```rust
// Returns "some type implementing Iterator"
fn make_iter() -> impl Iterator<Item = i32> {
    (0..10).map(|x| x * 2).filter(|x| *x > 5)
}

// Without impl Trait, you'd need this monster:
// fn make_iter() -> Filter<Map<Range<i32>, fn(i32) -> i32>, fn(&i32) -> bool>
```

## In Argument Position

```rust
// These are equivalent:
fn print_items(iter: impl Iterator<Item = i32>) {
    for i in iter {
        println!("{i}");
    }
}

fn print_items<I: Iterator<Item = i32>>(iter: I) {
    for i in iter {
        println!("{i}");
    }
}
```

## impl Trait vs dyn Trait

| Feature | `impl Trait` | `dyn Trait` |
|---------|--------------|-------------|
| Dispatch | Static | Dynamic |
| Size known | At compile time | At runtime |
| Heterogeneous | No | Yes |
| Performance | Fastest | Vtable overhead |

## Limitations

```rust
// Can't return different concrete types:
fn bad(cond: bool) -> impl Display {
    if cond {
        42      // i32
    } else {
        "hi"   // &str  - Error!
    }
}

// Use dyn Trait for that:
fn good(cond: bool) -> Box<dyn Display> {
    if cond {
        Box::new(42)
    } else {
        Box::new("hi")
    }
}
```

## Code Examples

**impl Trait usage**

```rust
// Returning closures
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n
}

let add_five = make_adder(5);
println!("{}", add_five(10));  // 15

// Returning iterators
fn evens(max: i32) -> impl Iterator<Item = i32> {
    (0..max).filter(|n| n % 2 == 0)
}

// Type alias with impl Trait (Rust 1.75+)
type BoxedIterator = impl Iterator<Item = i32>;

fn create_iter() -> BoxedIterator {
    (0..100).filter(|x| x % 2 == 0)
}
```


---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*