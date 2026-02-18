---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-proc-macro-types"
---

# Three Types of Proc Macros

## 1. Custom Derive

```rust
#[derive(MyTrait)]  // Adds implementation
struct MyStruct { ... }
```

## 2. Attribute-Like

```rust
#[my_attribute]  // Can modify the item
fn my_function() { ... }
```

## 3. Function-Like

```rust
my_macro!(input);  // Called like macro_rules! but more powerful
```

# Project Structure

Proc macros must be in a separate crate:

```
my_project/
├── Cargo.toml
├── src/
│   └── lib.rs
└── my_project_derive/      # Proc macro crate
    ├── Cargo.toml
    └── src/
        └── lib.rs
```

## Proc Macro Crate Setup

```toml
# my_project_derive/Cargo.toml
[package]
name = "my_project_derive"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true  # Required!

[dependencies]
syn = "2"          # Parse Rust code
quote = "1"        # Generate Rust code
proc-macro2 = "1" # Better TokenStream
```

## Re-export from Main Crate

```rust
// my_project/src/lib.rs
pub use my_project_derive::MyDerive;

// Users just do:
// use my_project::MyDerive;
```

See [Procedural Macros](https://doc.rust-lang.org/reference/procedural-macros.html).

## Code Examples

**Three proc macro signatures**

```rust
// my_project_derive/src/lib.rs
use proc_macro::TokenStream;

// Custom derive
#[proc_macro_derive(MyTrait)]
pub fn my_trait_derive(input: TokenStream) -> TokenStream {
    // input: the struct/enum being derived
    // output: new impl block
    todo!()
}

// Attribute macro
#[proc_macro_attribute]
pub fn my_attribute(args: TokenStream, input: TokenStream) -> TokenStream {
    // args: the arguments in #[my_attribute(args)]
    // input: the item the attribute is on
    // output: modified or new item
    todo!()
}

// Function-like macro
#[proc_macro]
pub fn my_function_macro(input: TokenStream) -> TokenStream {
    // input: everything inside the parentheses
    // output: Rust code to insert
    todo!()
}
```


## Resources

- [Procedural Macros](https://doc.rust-lang.org/reference/procedural-macros.html) — Official proc macro reference

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*