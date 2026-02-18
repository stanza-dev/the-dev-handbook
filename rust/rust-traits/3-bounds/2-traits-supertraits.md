---
source_course: "rust-traits"
source_lesson: "rust-traits-supertraits"
---

# Supertraits

A trait can require other traits:

```rust
use std::fmt::Debug;

// Error requires Debug
trait Error: Debug {
    fn description(&self) -> &str;
}

// To implement Error, you must also implement Debug
#[derive(Debug)]
struct MyError;

impl Error for MyError {
    fn description(&self) -> &str {
        "my error"
    }
}
```

## Multiple Supertraits

```rust
trait MyTrait: Debug + Clone + Send {
    fn do_something(&self);
}
```

## Supertraits in std

```rust
// Copy requires Clone
trait Copy: Clone { }

// Eq requires PartialEq
trait Eq: PartialEq<Self> { }

// Error requires Debug + Display
trait Error: Debug + Display { }
```

# Blanket Implementations

Implement a trait for all types matching a bound:

```rust
// From std: anything that implements Display
// automatically implements ToString
impl<T: Display> ToString for T {
    fn to_string(&self) -> String {
        format!("{self}")
    }
}

// Now every Display type has .to_string()
let s = 42.to_string();
let s = 3.14.to_string();
```

## Common Blanket Impls

```rust
// From<T> gives you Into<T>
impl<T, U> Into<U> for T
where U: From<T>
{
    fn into(self) -> U {
        U::from(self)
    }
}

// Reference types
impl<T: ?Sized> AsRef<T> for T {
    fn as_ref(&self) -> &T {
        self
    }
}
```

## Code Examples

**Blanket implementation**

```rust
// Custom blanket implementation
use std::fmt::Debug;

trait Loggable: Debug {
    fn log(&self) {
        println!("[LOG] {:?}", self);
    }
}

// Blanket impl: all Debug types are Loggable
impl<T: Debug> Loggable for T {}

// Now everything Debug can .log()
fn main() {
    42.log();                    // [LOG] 42
    "hello".log();               // [LOG] "hello"
    vec![1, 2, 3].log();         // [LOG] [1, 2, 3]
    Some(true).log();            // [LOG] Some(true)
}
```


---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*