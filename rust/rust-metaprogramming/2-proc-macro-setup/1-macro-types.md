---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-proc-macro-types"
---

# Types of Procedural Macros

## Introduction
Procedural macros are Rust functions that operate on token streams at compile time, giving you the full power of Rust's type system and standard library during code generation. Unlike declarative macros that use pattern matching, proc macros are arbitrary Rust code that receives tokens as input and produces tokens as output. There are three distinct kinds, each suited to different use cases.

## Key Concepts
- **Procedural macro**: A Rust function compiled into a special crate that the compiler loads as a plugin during compilation.
- **Custom derive**: Generates additional implementations (usually trait impls) for a struct or enum annotated with `#[derive(MyTrait)]`.
- **Attribute macro**: Receives both the attribute's arguments and the annotated item, and can modify, replace, or augment that item.
- **Function-like macro**: Invoked with `my_macro!(...)` syntax, similar to declarative macros but with full programmatic control.
- **proc-macro crate**: A special crate type declared with `proc-macro = true` in Cargo.toml that can export proc macro functions.

## Real World Context
Procedural macros power many of Rust's most popular libraries. Serde uses `#[derive(Serialize, Deserialize)]` to generate serialization code. Tokio uses `#[tokio::main]` as an attribute macro to set up the async runtime. SQLx uses function-like macros to validate SQL queries at compile time. Understanding the three types and when to use each is essential for working with the Rust ecosystem.

## Deep Dive

### The Three Types

**1. Custom Derive** — adds code alongside the annotated item:

```rust
#[derive(MyTrait)]  // Generates: impl MyTrait for MyStruct { ... }
struct MyStruct {
    name: String,
    age: u32,
}
```

Derive macros can only *add* new items (typically trait implementations). They cannot modify the original struct.

**2. Attribute-Like** — can transform the annotated item entirely:

```rust
#[my_attribute(option = "value")]  // Can modify or replace the function
fn my_function() { /* ... */ }
```

Attribute macros receive two token streams: the arguments inside the attribute and the item being annotated. They return a new token stream that replaces the original item.

**3. Function-Like** — called with bang syntax, accepts arbitrary tokens:

```rust
let query = sql!("SELECT * FROM users WHERE active = true");
```

Function-like macros look like declarative macro calls but can perform complex parsing and validation that `macro_rules!` cannot.

### Project Structure

Proc macros must live in a separate crate. Here is the typical layout:

```
my_project/
├── Cargo.toml           # Main crate, depends on my_project_derive
├── src/
│   └── lib.rs           # Re-exports derive macros for convenience
└── my_project_derive/   # Proc macro crate
    ├── Cargo.toml       # proc-macro = true
    └── src/
        └── lib.rs       # Macro implementations
```

This separation is required because proc macros run at compile time and need their own compilation unit.

### Cargo.toml for the Proc Macro Crate

The proc macro crate's Cargo.toml must declare `proc-macro = true`:

```toml
[package]
name = "my_project_derive"
version = "0.1.0"
edition = "2024"

[lib]
proc-macro = true  # Required: marks this as a proc macro crate

[dependencies]
syn = "2"          # Parse Rust tokens into an AST
quote = "1"        # Generate Rust tokens from quasi-quoted templates
proc-macro2 = "1" # Improved TokenStream type for testing and composition
```

The three key dependencies (`syn`, `quote`, `proc-macro2`) form the standard proc macro toolkit. `syn` parses input tokens, `quote` generates output tokens, and `proc-macro2` bridges them together.

### Re-exporting from the Main Crate

The main crate typically re-exports the derive macros so users have a single dependency:

```rust
// my_project/src/lib.rs
pub use my_project_derive::MyDerive;

// Users write:
// use my_project::MyDerive;
// instead of:
// use my_project_derive::MyDerive;
```

This pattern is used by serde, clap, and most other Rust libraries that provide derive macros.

## Common Pitfalls
1. **Forgetting `proc-macro = true`** — Without this flag in Cargo.toml, the compiler treats the crate as a normal library and `#[proc_macro_derive]` attributes produce errors.
2. **Putting proc macro code in the main crate** — Proc macros must be in a separate crate. You cannot define a `#[proc_macro_derive]` function in a regular `lib.rs`.
3. **Using the wrong syn version** — syn 2.0 has breaking changes from syn 1.x. Always use `syn = "2"` for new projects. Notable removals include `AttributeArgs` and `NestedMeta` types.

## Best Practices
1. **Name the derive crate `{project}_derive` or `{project}_macros`** — This is the established convention (e.g., `serde_derive`, `tokio_macros`).
2. **Re-export from the main crate** — Users should not need to add the derive crate as a direct dependency.
3. **Keep macro logic testable** — Extract the core transformation into a function that takes `proc_macro2::TokenStream` and returns `proc_macro2::TokenStream`, so you can write unit tests without the compiler plugin infrastructure.

## Summary
- Procedural macros are Rust functions that transform tokens at compile time.
- The three types are: custom derive, attribute-like, and function-like.
- Proc macros must live in a crate with `proc-macro = true` in Cargo.toml.
- The standard toolkit is `syn` 2 + `quote` 1 + `proc-macro2` 1.
- Re-export macros from the main crate for a clean user API.

## Code Examples

**The three proc macro function signatures — each receives and returns TokenStream but with different semantics**

```rust
// my_project_derive/src/lib.rs
use proc_macro::TokenStream;

// 1. Custom derive: generates an impl block
#[proc_macro_derive(MyTrait)]
pub fn my_trait_derive(input: TokenStream) -> TokenStream {
    // input: the struct/enum tokens
    // output: new impl block added alongside the item
    todo!()
}

// 2. Attribute macro: can modify the annotated item
#[proc_macro_attribute]
pub fn my_attribute(args: TokenStream, input: TokenStream) -> TokenStream {
    // args: tokens inside #[my_attribute(these_tokens)]
    // input: the item the attribute is attached to
    // output: replacement for the original item
    todo!()
}

// 3. Function-like macro: invoked as my_macro!(...)
#[proc_macro]
pub fn my_function_macro(input: TokenStream) -> TokenStream {
    // input: everything between the parentheses
    // output: Rust code to substitute in place
    todo!()
}
```


## Resources

- [Procedural Macros - The Rust Reference](https://doc.rust-lang.org/reference/procedural-macros.html) — Official specification for procedural macro types, crate requirements, and hygiene rules

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*