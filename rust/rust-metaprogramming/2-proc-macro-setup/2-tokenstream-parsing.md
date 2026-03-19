---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-tokenstream-parsing"
---

# TokenStreams & syn Parsing

## Introduction
At their core, procedural macros receive and produce `TokenStream` values — flat sequences of tokens that represent Rust source code. The `syn` crate transforms these raw tokens into a structured abstract syntax tree (AST) that you can inspect and manipulate programmatically. Understanding the TokenStream-to-AST pipeline is the foundation of all proc macro work.

## Key Concepts
- **TokenStream**: A sequence of token trees (identifiers, punctuation, literals, and delimited groups) that represents Rust source code.
- **Token tree**: The atomic unit of a TokenStream — either a single token or a balanced group wrapped in `()`, `[]`, or `{}`.
- **`syn::DeriveInput`**: The primary AST type for derive macros, containing the name, generics, attributes, and data (struct fields or enum variants) of the annotated type.
- **`parse_macro_input!`**: A convenience macro from `syn` that parses a `TokenStream` into a specified AST type, automatically generating a compile error on parse failure.

## Real World Context
Every derive macro in the Rust ecosystem goes through this same pipeline: receive `TokenStream`, parse with `syn`, inspect the AST, generate code with `quote`, and return a new `TokenStream`. Libraries like serde_derive parse `DeriveInput` to read struct fields and generate serialization code. Understanding `DeriveInput` and its `Data` enum lets you write macros that handle structs, enums, and unions correctly.

## Deep Dive

### TokenStream Basics

A `TokenStream` is a flat list of tokens. The Rust source `fn greet() {}` becomes the token sequence `[fn, greet, (), {}]`:

```rust
use proc_macro::TokenStream;

// TokenStream contains four kinds of token trees:
// - Ident: keywords and names (fn, greet, struct, i32)
// - Punct: punctuation characters (+, -, ::, =>)
// - Literal: values (42, "hello", 3.14)
// - Group: balanced delimiters with contents ((), [], {})
```

You rarely work with raw tokens directly. Instead, `syn` provides structured types.

### Parsing with syn

The `parse_macro_input!` macro converts a `TokenStream` into any type that implements `syn::parse::Parse`:

```rust
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(MyDerive)]
pub fn my_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);

    // Now you have structured access:
    // input.ident    — the type name (e.g., "Person")
    // input.generics — generic parameters (<T, U>)
    // input.data     — the fields or variants
    // input.attrs    — attributes on the type
    // input.vis      — visibility (pub, pub(crate), etc.)

    todo!()
}
```

If the input tokens cannot be parsed as a `DeriveInput`, the macro automatically emits a helpful compile error pointing to the problematic location.

### The DeriveInput Structure

The `DeriveInput` type represents any item that can appear after `#[derive(...)]`:

```rust
pub struct DeriveInput {
    pub attrs: Vec<Attribute>,  // #[...] attributes on the type
    pub vis: Visibility,        // pub, pub(crate), or private
    pub ident: Ident,           // The type name
    pub generics: Generics,     // <T: Clone, U>
    pub data: Data,             // Struct fields, enum variants, or union
}

pub enum Data {
    Struct(DataStruct),  // struct Foo { ... }
    Enum(DataEnum),      // enum Bar { ... }
    Union(DataUnion),    // union Baz { ... }
}
```

The `Data` enum branches on whether the annotated item is a struct, enum, or union. Each variant gives access to the fields or variants.

### Extracting Struct Fields

Most derive macros need to iterate over a struct's fields:

```rust
use syn::{Data, Fields};

match &input.data {
    Data::Struct(data) => {
        match &data.fields {
            Fields::Named(fields) => {
                // struct Foo { bar: i32, baz: String }
                for field in &fields.named {
                    let name = &field.ident; // Some("bar"), Some("baz")
                    let ty = &field.ty;       // i32, String
                }
            }
            Fields::Unnamed(fields) => {
                // struct Foo(i32, String)
                for (index, field) in fields.unnamed.iter().enumerate() {
                    let ty = &field.ty; // Access by index, no names
                }
            }
            Fields::Unit => {
                // struct Foo; (no fields)
            }
        }
    }
    Data::Enum(data) => {
        for variant in &data.variants {
            let variant_name = &variant.ident;
            // Each variant has its own Fields
        }
    }
    Data::Union(_) => {
        // Unions are rare; most macros reject them
    }
}
```

Each `Field` object contains the field's name (if named), type, visibility, and any attributes.

## Common Pitfalls
1. **Forgetting to handle enums** — If your derive macro only matches `Data::Struct`, users who apply it to an enum will get a panic. Either handle enums or return a clear compile error with `syn::Error`.
2. **Unwrapping field ident on tuple structs** — `field.ident` is `None` for tuple struct fields. Use `.as_ref()` or match on `Fields::Named` vs `Fields::Unnamed` explicitly.
3. **Ignoring generics** — If you generate an `impl` block but forget to include the type's generic parameters, the code will fail to compile for generic types.

## Best Practices
1. **Use `syn::Error` instead of `panic!`** — Return `syn::Error::new_spanned(item, "message")` to produce a compile error that points to the offending code, rather than a panic that produces an unhelpful error.
2. **Test parsing with `syn::parse2`** — In unit tests, use `syn::parse2::<DeriveInput>(tokens)` with `proc_macro2::TokenStream` to test your macro logic without the compiler plugin runtime.
3. **Handle all three field types** — Named, unnamed, and unit structs have different access patterns. A robust macro handles or rejects each explicitly.

## Summary
- `TokenStream` is a flat sequence of tokens; `syn` parses it into structured AST types.
- `DeriveInput` is the main type for derive macros, giving access to name, generics, attributes, and fields.
- The `Data` enum distinguishes structs, enums, and unions.
- Use `parse_macro_input!` for convenient parsing with automatic error reporting.
- Always handle or explicitly reject enums, tuple structs, and generics.

## Code Examples

**A complete derive macro that extracts field names using syn's DeriveInput with proper error handling for unsupported types**

```rust
use syn::{parse_macro_input, DeriveInput, Data, Fields, Error};
use quote::quote;
use proc_macro::TokenStream;

// Derive macro that generates a field_names() method
#[proc_macro_derive(FieldNames)]
pub fn field_names_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;

    let field_names: Vec<_> = match &input.data {
        Data::Struct(data) => match &data.fields {
            Fields::Named(fields) => {
                fields.named.iter()
                    .map(|f| f.ident.as_ref().unwrap().to_string())
                    .collect()
            }
            _ => return Error::new_spanned(
                &input.ident,
                "FieldNames only supports structs with named fields"
            ).to_compile_error().into(),
        },
        _ => return Error::new_spanned(
            &input.ident,
            "FieldNames only supports structs"
        ).to_compile_error().into(),
    };

    let output = quote! {
        impl #name {
            pub fn field_names() -> &'static [&'static str] {
                &[#(#field_names),*]
            }
        }
    };

    output.into()
}

// Usage:
// #[derive(FieldNames)]
// struct User { name: String, email: String, age: u32 }
// User::field_names() => ["name", "email", "age"]
```


## Resources

- [syn crate documentation](https://docs.rs/syn/2/syn/) — API reference for syn 2.0, including DeriveInput, Data, Fields, and parsing utilities

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*