---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-attribute-args"
---

# Parsing Arguments

## Simple Arguments

```rust
// #[my_attr(foo, bar)]
let args = parse_macro_input!(args as syn::AttributeArgs);

for arg in args {
    match arg {
        NestedMeta::Meta(Meta::Path(path)) => {
            // Simple identifier: foo, bar
            let name = path.get_ident();
        }
        NestedMeta::Meta(Meta::NameValue(nv)) => {
            // key = value
            let key = &nv.path;
            let value = &nv.lit;
        }
        NestedMeta::Lit(lit) => {
            // Literal: "string", 42
        }
    }
}
```

## Custom Parsing

```rust
use syn::parse::{Parse, ParseStream};

struct MyArgs {
    name: syn::Ident,
    count: syn::LitInt,
}

impl Parse for MyArgs {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let name: syn::Ident = input.parse()?;
        input.parse::<syn::Token![,]>()?;
        let count: syn::LitInt = input.parse()?;
        Ok(MyArgs { name, count })
    }
}

#[proc_macro_attribute]
pub fn my_attr(args: TokenStream, input: TokenStream) -> TokenStream {
    let MyArgs { name, count } = parse_macro_input!(args as MyArgs);
    // Use name and count...
    todo!()
}

// Usage: #[my_attr(foo, 42)]
```

## Helper Attributes in Derive

```rust
#[proc_macro_derive(MyDerive, attributes(my_field_attr))]
pub fn my_derive(input: TokenStream) -> TokenStream {
    // Can now recognize #[my_field_attr(...)] on fields
}

#[derive(MyDerive)]
struct Example {
    #[my_field_attr(skip)]
    internal: i32,
    
    #[my_field_attr(rename = "other_name")]
    name: String,
}
```

## Code Examples

**Derive with serde-like attributes**

```rust
// Derive with helper attributes
use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput, Data, Fields, Attribute};
use quote::quote;

#[proc_macro_derive(Serialize, attributes(serde))]
pub fn serialize_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    
    let fields = match &input.data {
        Data::Struct(data) => match &data.fields {
            Fields::Named(f) => &f.named,
            _ => panic!("Only named fields supported"),
        },
        _ => panic!("Only structs supported"),
    };
    
    let field_serializers = fields.iter().map(|f| {
        let field_name = &f.ident;
        let serialized_name = get_serde_rename(&f.attrs)
            .unwrap_or_else(|| field_name.as_ref().unwrap().to_string());
        
        if has_serde_skip(&f.attrs) {
            quote! {}  // Skip this field
        } else {
            quote! {
                map.insert(#serialized_name.to_string(), 
                          format!("{:?}", self.#field_name));
            }
        }
    });
    
    let output = quote! {
        impl #name {
            pub fn serialize(&self) -> std::collections::HashMap<String, String> {
                let mut map = std::collections::HashMap::new();
                #(#field_serializers)*
                map
            }
        }
    };
    
    output.into()
}

fn get_serde_rename(attrs: &[Attribute]) -> Option<String> {
    // Parse #[serde(rename = "...")]
    for attr in attrs {
        if attr.path.is_ident("serde") {
            // Parse nested attributes...
        }
    }
    None
}

fn has_serde_skip(attrs: &[Attribute]) -> bool {
    // Check for #[serde(skip)]
    false
}
```


---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*