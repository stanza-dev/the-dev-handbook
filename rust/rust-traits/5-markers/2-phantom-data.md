---
source_course: "rust-traits"
source_lesson: "rust-traits-phantom-data"
---

# PhantomData: Zero-Cost Type Parameters

`PhantomData` lets you "use" a type parameter without storing it:

```rust
use std::marker::PhantomData;

// Without PhantomData:
struct Wrapper<T> {  // Error! T is unused
    value: i32,
}

// With PhantomData:
struct Wrapper<T> {
    value: i32,
    _marker: PhantomData<T>,  // "Uses" T with zero cost
}
```

## Why Use PhantomData?

### 1. Variance Control

```rust
// PhantomData<T> makes struct covariant in T
// PhantomData<fn(T)> makes struct contravariant in T
// PhantomData<*mut T> makes struct invariant in T

struct Covariant<T> {
    _marker: PhantomData<T>,  // Covariant
}

struct Invariant<T> {
    _marker: PhantomData<*mut T>,  // Invariant
}
```

### 2. Drop Check

```rust
// Tell the compiler we "own" T
struct Container<T> {
    ptr: *mut T,
    _marker: PhantomData<T>,  // Indicates ownership
}

impl<T> Drop for Container<T> {
    fn drop(&mut self) {
        // Safe to drop T because PhantomData indicates we own it
    }
}
```

### 3. Lifetime Tracking

```rust
struct Iter<'a, T> {
    ptr: *const T,
    end: *const T,
    _marker: PhantomData<&'a T>,  // Tracks lifetime 'a
}
```

### 4. Type Tags

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

// Can't mix units!
let d1: Distance<Meters> = Distance::new(100.0);
let d2: Distance<Feet> = Distance::new(328.0);
// d1 + d2  // Error! Different types
```

See [PhantomData](https://doc.rust-lang.org/std/marker/struct.PhantomData.html).

## Code Examples

**Type-safe builder with PhantomData**

```rust
use std::marker::PhantomData;

// Builder pattern with type state
struct Builder<T, HasName, HasAge> {
    name: Option<String>,
    age: Option<u32>,
    _marker: PhantomData<(T, HasName, HasAge)>,
}

struct Yes;
struct No;

impl<T> Builder<T, No, No> {
    fn new() -> Self {
        Builder {
            name: None,
            age: None,
            _marker: PhantomData,
        }
    }
}

impl<T, Age> Builder<T, No, Age> {
    fn name(self, name: &str) -> Builder<T, Yes, Age> {
        Builder {
            name: Some(name.to_string()),
            age: self.age,
            _marker: PhantomData,
        }
    }
}

impl<T, Name> Builder<T, Name, No> {
    fn age(self, age: u32) -> Builder<T, Name, Yes> {
        Builder {
            name: self.name,
            age: Some(age),
            _marker: PhantomData,
        }
    }
}

// Only buildable when both fields are set!
impl<T> Builder<T, Yes, Yes> {
    fn build(self) -> (String, u32) {
        (self.name.unwrap(), self.age.unwrap())
    }
}
```


## Resources

- [PhantomData](https://doc.rust-lang.org/std/marker/struct.PhantomData.html) — PhantomData documentation

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*