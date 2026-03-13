---
source_course: "rust-traits"
source_lesson: "rust-traits-phantom-data"
---

# PhantomData

## Introduction
`PhantomData` is a zero-sized type that tells the compiler a struct "uses" a type parameter without actually storing it. It is essential for controlling variance, indicating ownership, tracking lifetimes, and creating type-level tags.

## Key Concepts
- **PhantomData<T>**: A zero-sized marker that makes the compiler treat the struct as if it contains a `T`.
- **Variance**: How a type's subtyping relationships change with its parameters. `PhantomData<T>` makes the struct covariant in `T`.
- **Type Tag**: A zero-cost type parameter that distinguishes otherwise identical structs at the type level.

## Real World Context
Every custom smart pointer, iterator, and unsafe abstraction in Rust uses `PhantomData`. The `Iter<'a, T>` type in the standard library uses `PhantomData<&'a T>` to track its lifetime. Unit-of-measure patterns use `PhantomData` to prevent mixing incompatible units.

## Deep Dive

### Basic Usage

```rust
use std::marker::PhantomData;

// Without PhantomData: error — T is unused
// struct Wrapper<T> { value: i32 }

// With PhantomData: compiles
struct Wrapper<T> {
    value: i32,
    _marker: PhantomData<T>,
}
```

### Unit-of-Measure Pattern

```rust
struct Meters;
struct Feet;

struct Distance<Unit> {
    value: f64,
    _unit: PhantomData<Unit>,
}

impl<U> Distance<U> {
    fn new(value: f64) -> Self {
        Distance { value, _unit: PhantomData }
    }
}

let d1: Distance<Meters> = Distance::new(100.0);
let d2: Distance<Feet> = Distance::new(328.0);
// d1 + d2 would be a type error — different units!
```

### Variance Control

```rust
// Covariant in T (default with PhantomData<T>)
struct Covariant<T> { _m: PhantomData<T> }

// Invariant in T (use PhantomData<fn(T) -> T> or PhantomData<*mut T>)
struct Invariant<T> { _m: PhantomData<fn(T) -> T> }
```

### Lifetime Tracking

```rust
struct Iter<'a, T> {
    ptr: *const T,
    end: *const T,
    _marker: PhantomData<&'a T>, // Tracks lifetime 'a
}
```

## Common Pitfalls
1. **Forgetting PhantomData** — Unused type parameters cause a compile error. Always add `PhantomData` when a type parameter is not stored directly.
2. **Wrong variance** — `PhantomData<T>` makes the struct covariant in `T` and tells the compiler the struct logically owns a `T` (affecting drop check). For shared references use `PhantomData<&'a T>`, for invariance use `PhantomData<fn(T) -> T>`.

## Best Practices
1. **Name the field `_marker` or `_phantom`** — The underscore prefix signals it is intentionally unused at runtime.
2. **Choose the right PhantomData form for your variance needs** — `PhantomData<T>` for ownership, `PhantomData<&'a T>` for borrowing, `PhantomData<fn(T) -> T>` for invariance.

## Summary
- `PhantomData<T>` is zero-sized and tells the compiler the struct logically contains `T`.
- Use it for type tags (units of measure), lifetime tracking, variance control, and drop check.
- Different `PhantomData` forms control variance: `<T>` for covariant, `<fn(T)->T>` for invariant.
- It is essential for any unsafe abstraction that works with raw pointers or lifetimes.

## Code Examples

**PhantomData creates type-safe IDs that prevent mixing different entity types at compile time, with zero runtime overhead**

```rust
use std::marker::PhantomData;

// Type-safe ID pattern — prevents mixing User IDs with Post IDs
struct Id<T> {
    value: u64,
    _marker: PhantomData<T>,
}

impl<T> Id<T> {
    fn new(value: u64) -> Self {
        Id { value, _marker: PhantomData }
    }
}

struct User;
struct Post;

let user_id: Id<User> = Id::new(1);
let post_id: Id<Post> = Id::new(1);

// fn get_user(id: Id<User>) { ... }
// get_user(post_id); // Compile error! Expected Id<User>, got Id<Post>

// Same underlying u64, but the type system prevents mixing them
assert_eq!(std::mem::size_of::<Id<User>>(), 8); // PhantomData is zero-sized
```


## Resources

- [PhantomData](https://doc.rust-lang.org/std/marker/struct.PhantomData.html) — Official Rust documentation for PhantomData and its use cases

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*