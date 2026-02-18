---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-repetition-hygiene"
---

# Repetition in Macros

```rust
// Syntax: $( PATTERN )SEPARATOR QUANTIFIER
// QUANTIFIER: * (0+), + (1+), ? (0 or 1)

macro_rules! make_struct {
    ($name:ident { $($field:ident: $ty:ty),* }) => {
        struct $name {
            $( $field: $ty, )*
        }
    };
}

make_struct!(Person { name: String, age: u32 });
// Expands to:
// struct Person {
//     name: String,
//     age: u32,
// }
```

## Nested Repetition

```rust
macro_rules! nested {
    ($($outer:ident: [$($inner:expr),*]);*) => {
        $(
            let $outer = vec![$($inner),*];
        )*
    };
}

nested!(a: [1, 2, 3]; b: [4, 5]);
// Creates:
// let a = vec![1, 2, 3];
// let b = vec![4, 5];
```

# Macro Hygiene

Macros have their own "scope" to prevent variable name clashes:

```rust
macro_rules! make_x {
    () => {
        let x = 42;  // This x is "hygienic"
    };
}

fn main() {
    let x = 1;
    make_x!();
    println!("{x}");  // Still 1! Macro's x is different
}
```

## Breaking Hygiene (When Needed)

```rust
macro_rules! with_x {
    ($body:block) => {
        {
            let x = 42;
            $body  // $body can see x because it's from caller
        }
    };
}

with_x!({ println!("{x}"); });  // Prints 42
```

## The tt Muncher Pattern

```rust
macro_rules! count {
    () => { 0 };
    ($head:tt $($tail:tt)*) => {
        1 + count!($($tail)*)
    };
}

assert_eq!(count!(a b c d), 4);
```

## Code Examples

**HashMap literal macro**

```rust
// HashMap literal macro
macro_rules! hashmap {
    () => {
        ::std::collections::HashMap::new()
    };
    ($($key:expr => $value:expr),+ $(,)?) => {
        {
            let mut map = ::std::collections::HashMap::new();
            $(
                map.insert($key, $value);
            )+
            map
        }
    };
}

let empty: std::collections::HashMap<i32, i32> = hashmap!{};
let scores = hashmap!{
    "Alice" => 100,
    "Bob" => 95,
    "Charlie" => 87,
};

assert!(empty.is_empty());
assert_eq!(scores.get("Alice"), Some(&100));
assert_eq!(scores.len(), 3);
```


---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*