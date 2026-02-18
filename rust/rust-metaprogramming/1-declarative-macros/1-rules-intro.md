---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-macro-rules-intro"
---

# Declarative Macros with macro_rules!

Pattern-matching macros that transform code at compile time.

```rust
macro_rules! say_hello {
    () => {
        println!("Hello!");
    };
}

say_hello!();  // Expands to: println!("Hello!");
```

## Fragment Specifiers

Capture different types of syntax:

| Specifier | Matches |
|-----------|----------|
| `expr` | Expressions |
| `ident` | Identifiers |
| `ty` | Types |
| `pat` | Patterns |
| `path` | Paths (std::io::Result) |
| `stmt` | Statements |
| `block` | Blocks { ... } |
| `item` | Items (fn, struct, etc.) |
| `literal` | Literals (42, "hi") |
| `tt` | Token tree (anything) |
| `meta` | Attributes content |
| `lifetime` | Lifetimes ('a) |
| `vis` | Visibility (pub, pub(crate)) |

## Basic Pattern Matching

```rust
macro_rules! create_function {
    ($name:ident) => {
        fn $name() {
            println!("Called {}", stringify!($name));
        }
    };
}

create_function!(foo);  // Creates fn foo()
create_function!(bar);  // Creates fn bar()

foo();  // "Called foo"
bar();  // "Called bar"
```

## Multiple Arms

```rust
macro_rules! print_type {
    ($val:expr) => {
        println!("{:?}", $val);
    };
    ($val:expr, $label:literal) => {
        println!("{}: {:?}", $label, $val);
    };
}

print_type!(42);             // "42"
print_type!(42, "value");    // "value: 42"
```

See [Macros by Example](https://doc.rust-lang.org/reference/macros-by-example.html).

## Code Examples

**Recreating vec! macro**

```rust
// Recreating vec! macro
macro_rules! my_vec {
    // Empty vec
    () => {
        Vec::new()
    };
    // vec![1, 2, 3]
    ($($elem:expr),+ $(,)?) => {
        {
            let mut v = Vec::new();
            $(
                v.push($elem);
            )+
            v
        }
    };
}

let v1: Vec<i32> = my_vec![];
let v2 = my_vec![1, 2, 3];
let v3 = my_vec![1, 2, 3,];  // Trailing comma OK

assert!(v1.is_empty());
assert_eq!(v2, vec![1, 2, 3]);
assert_eq!(v3, vec![1, 2, 3]);
```


## Resources

- [Macros by Example](https://doc.rust-lang.org/reference/macros-by-example.html) — Official macro_rules! reference

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*