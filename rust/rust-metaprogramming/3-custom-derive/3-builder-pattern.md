---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-derive-builder-pattern"
---

# Builder Pattern Derive

## Introduction
The builder pattern is one of the most popular use cases for custom derive macros. Instead of writing repetitive setter methods by hand for each field, a `#[derive(Builder)]` macro can generate the entire builder struct, its setter methods, and a `build()` method that validates and constructs the target type. This lesson walks through a complete, production-quality builder derive.

## Key Concepts
- **Builder pattern**: A creational pattern where an object is constructed step-by-step through setter methods, with validation at the final `build()` step.
- **`Option<T>` wrapping**: The generated builder struct wraps each field in `Option<T>`, starting as `None` and filling in as setters are called.
- **`format_ident!`**: A `quote` macro that creates new identifiers by combining strings, useful for generating names like `PersonBuilder` from `Person`.

## Real World Context
The builder pattern is used throughout Rust for constructing complex objects: `reqwest::Client::builder()`, `tokio::runtime::Builder`, `clap::Command::new()`. The `derive_builder` crate on crates.io provides this exact functionality. Building it yourself is one of the best exercises for learning proc macros because it touches field iteration, type manipulation, and code generation all at once.

## Deep Dive

### Step 1: Define the Builder Struct

For each field in the target struct, the builder needs a corresponding `Option<T>` field:

```rust
let builder_name = format_ident!("{}Builder", name);

let builder_fields = fields.iter().map(|f| {
    let field_name = &f.ident;
    let field_type = &f.ty;
    quote! { #field_name: Option<#field_type> }
});

let builder_defaults = fields.iter().map(|f| {
    let field_name = &f.ident;
    quote! { #field_name: None }
});
```

The `format_ident!` macro creates a new identifier by appending "Builder" to the struct name. Each field becomes `Option<OriginalType>` initialized to `None`.

### Step 2: Generate Setter Methods

Each setter takes a value, wraps it in `Some`, and returns `self` for chaining:

```rust
let setters = fields.iter().map(|f| {
    let field_name = &f.ident;
    let field_type = &f.ty;
    quote! {
        pub fn #field_name(mut self, value: #field_type) -> Self {
            self.#field_name = Some(value);
            self
        }
    }
});
```

This generates methods like `fn name(mut self, value: String) -> Self` that the user chains together.

### Step 3: Generate the build() Method

The `build()` method unwraps each `Option` field, returning an error if any are missing:

```rust
let build_fields = fields.iter().map(|f| {
    let field_name = &f.ident;
    let err_msg = format!("{} is required", field_name.as_ref().unwrap());
    quote! {
        #field_name: self.#field_name.ok_or(#err_msg)?
    }
});
```

This produces `name: self.name.ok_or("name is required")?` for each field, using the `?` operator to return early on missing fields.

### Complete Assembly

Putting it all together, the macro generates a builder struct, a constructor on the original type, setter methods, and a build method:

```rust
quote! {
    pub struct #builder_name {
        #(#builder_fields,)*
    }

    impl #name {
        pub fn builder() -> #builder_name {
            #builder_name {
                #(#builder_defaults,)*
            }
        }
    }

    impl #builder_name {
        #(#setters)*

        pub fn build(self) -> Result<#name, String> {
            Ok(#name {
                #(#build_fields,)*
            })
        }
    }
}
```

Users can now write:

```rust
let server = ServerConfig::builder()
    .host("localhost".to_string())
    .port(8080)
    .max_connections(100)
    .build()?;  // Returns Err if any field was not set
```

## Common Pitfalls
1. **Forgetting to handle `Option<T>` fields** — If the original struct already has `Option<T>` fields, wrapping them again as `Option<Option<T>>` is awkward. Production builders detect this and make those fields optional in the builder.
2. **Not supporting `Default` values** — Some fields have sensible defaults. A production builder combines helper attributes like `#[builder(default)]` with the builder pattern.
3. **Consuming self in setters** — Taking `self` by value (not `&mut self`) means the builder cannot be reused. This is the common pattern in Rust, but document it clearly.

## Best Practices
1. **Return `Result` from `build()`** — Rather than panicking on missing fields, return a `Result` with a descriptive error message.
2. **Add helper attributes for customization** — Support `#[builder(default = "value")]` and `#[builder(setter(into))]` for real-world flexibility.
3. **Generate documentation** — Use `#[doc = "..."]` in the generated code so the builder methods appear in rustdoc.

## Summary
- The builder pattern is a natural fit for derive macros, eliminating field-by-field boilerplate.
- Fields are wrapped in `Option<T>` in the builder, starting as `None`.
- `format_ident!` creates derived identifiers like `PersonBuilder` from `Person`.
- The `build()` method validates that all required fields are set.
- Production builders add helper attributes for defaults and type conversions.

## Code Examples

**A complete Builder derive macro generating a builder struct with Option fields, chainable setters, and a validating build() method**

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput, Data, Fields, Error};
use quote::{quote, format_ident};

#[proc_macro_derive(Builder)]
pub fn builder_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    match generate_builder(&input) {
        Ok(tokens) => tokens.into(),
        Err(err) => err.to_compile_error().into(),
    }
}

fn generate_builder(input: &DeriveInput) -> Result<proc_macro2::TokenStream, Error> {
    let name = &input.ident;
    let builder_name = format_ident!("{}Builder", name);

    let fields = match &input.data {
        Data::Struct(data) => match &data.fields {
            Fields::Named(f) => &f.named,
            _ => return Err(Error::new_spanned(name, "Builder requires named fields")),
        },
        _ => return Err(Error::new_spanned(name, "Builder only works on structs")),
    };

    let builder_fields = fields.iter().map(|f| {
        let fname = &f.ident;
        let ftype = &f.ty;
        quote! { #fname: Option<#ftype> }
    });

    let defaults = fields.iter().map(|f| {
        let fname = &f.ident;
        quote! { #fname: None }
    });

    let setters = fields.iter().map(|f| {
        let fname = &f.ident;
        let ftype = &f.ty;
        quote! {
            pub fn #fname(mut self, value: #ftype) -> Self {
                self.#fname = Some(value);
                self
            }
        }
    });

    let build_fields = fields.iter().map(|f| {
        let fname = &f.ident;
        let err = format!("{} is required", fname.as_ref().unwrap());
        quote! { #fname: self.#fname.ok_or(#err)? }
    });

    Ok(quote! {
        pub struct #builder_name {
            #(#builder_fields,)*
        }
        impl #name {
            pub fn builder() -> #builder_name {
                #builder_name { #(#defaults,)* }
            }
        }
        impl #builder_name {
            #(#setters)*
            pub fn build(self) -> Result<#name, String> {
                Ok(#name { #(#build_fields,)* })
            }
        }
    })
}

// Usage:
// #[derive(Builder)]
// struct ServerConfig { host: String, port: u16, max_connections: usize }
//
// let config = ServerConfig::builder()
//     .host("localhost".into())
//     .port(8080)
//     .max_connections(100)
//     .build()?;
```


## Resources

- [derive_builder crate](https://docs.rs/derive_builder/latest/derive_builder/) — A production-quality builder derive crate demonstrating advanced patterns including defaults, validations, and custom setters

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*