---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-tokenstream-parsing"
---

# TokenStream

Proc macros work with `TokenStream` - a stream of tokens:

```rust
use proc_macro::TokenStream;

// TokenStream is: ident, punct, literal, group
// "fn foo() {}" becomes: [fn, foo, (), {}]
```

# syn: Parse TokenStream to AST

```rust
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(MyDerive)]
pub fn my_derive(input: TokenStream) -> TokenStream {
    // Parse into structured AST
    let input = parse_macro_input!(input as DeriveInput);
    
    // Now you have:
    // input.ident - the type name
    // input.generics - generic parameters
    // input.data - struct fields or enum variants
    
    todo!()
}
```

## DeriveInput Structure

```rust
pub struct DeriveInput {
    pub attrs: Vec<Attribute>,
    pub vis: Visibility,
    pub ident: Ident,
    pub generics: Generics,
    pub data: Data,  // Struct, Enum, or Union
}

pub enum Data {
    Struct(DataStruct),
    Enum(DataEnum),
    Union(DataUnion),
}
```

## Extracting Struct Fields

```rust
use syn::{Data, Fields};

match &input.data {
    Data::Struct(data) => {
        match &data.fields {
            Fields::Named(fields) => {
                for field in &fields.named {
                    let name = &field.ident;
                    let ty = &field.ty;
                    // Process each field
                }
            }
            Fields::Unnamed(fields) => { /* tuple struct */ }
            Fields::Unit => { /* unit struct */ }
        }
    }
    Data::Enum(data) => { /* enum variants */ }
    Data::Union(_) => { /* union */ }
}
```

## Code Examples

**Derive macro extracting field names**

```rust
use syn::{parse_macro_input, DeriveInput, Data, Fields};
use quote::quote;
use proc_macro::TokenStream;

#[proc_macro_derive(FieldNames)]
pub fn field_names_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    
    // Extract field names
    let field_names: Vec<_> = match &input.data {
        Data::Struct(data) => {
            match &data.fields {
                Fields::Named(fields) => {
                    fields.named.iter()
                        .map(|f| f.ident.as_ref().unwrap().to_string())
                        .collect()
                }
                _ => vec![],
            }
        }
        _ => panic!("FieldNames only works on structs with named fields"),
    };
    
    // Generate implementation
    let output = quote! {
        impl #name {
            pub fn field_names() -> &'static [&'static str] {
                &[#(#field_names),*]
            }
        }
    };
    
    output.into()
}
```


---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*