---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-derive-implementation"
---

# Step-by-Step Derive Macro

Let's create `#[derive(Hello)]` that generates a `hello()` method.

## Step 1: Parse Input

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput};
use quote::quote;

#[proc_macro_derive(Hello)]
pub fn hello_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    let name_str = name.to_string();
    
    // Generate the impl
    let expanded = quote! {
        impl #name {
            pub fn hello(&self) {
                println!("Hello from {}!", #name_str);
            }
        }
    };
    
    expanded.into()
}
```

## Step 2: Handle Generics

```rust
#[proc_macro_derive(Hello)]
pub fn hello_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    
    // Split generics for impl
    let (impl_generics, ty_generics, where_clause) = 
        input.generics.split_for_impl();
    
    let expanded = quote! {
        impl #impl_generics #name #ty_generics #where_clause {
            pub fn hello(&self) {
                println!("Hello from {}!", stringify!(#name));
            }
        }
    };
    
    expanded.into()
}
```

## Step 3: Use It

```rust
use my_derive::Hello;

#[derive(Hello)]
struct Person<T> {
    name: String,
    data: T,
}

fn main() {
    let p = Person { name: "Alice".into(), data: 42 };
    p.hello();  // "Hello from Person!"
}
```

## Code Examples

**Custom Debug derive implementation**

```rust
// Complete example: Debug-like derive
use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput, Data, Fields};
use quote::quote;

#[proc_macro_derive(SimpleDebug)]
pub fn simple_debug_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    let (impl_generics, ty_generics, where_clause) = 
        input.generics.split_for_impl();
    
    let debug_body = match &input.data {
        Data::Struct(data) => {
            let fields = match &data.fields {
                Fields::Named(fields) => {
                    let field_prints = fields.named.iter().map(|f| {
                        let name = &f.ident;
                        quote! {
                            .field(stringify!(#name), &self.#name)
                        }
                    });
                    quote! {
                        f.debug_struct(stringify!(#name))
                            #(#field_prints)*
                            .finish()
                    }
                }
                Fields::Unnamed(fields) => {
                    let field_prints = (0..fields.unnamed.len()).map(|i| {
                        let index = syn::Index::from(i);
                        quote! { .field(&self.#index) }
                    });
                    quote! {
                        f.debug_tuple(stringify!(#name))
                            #(#field_prints)*
                            .finish()
                    }
                }
                Fields::Unit => {
                    quote! { f.write_str(stringify!(#name)) }
                }
            };
            fields
        }
        _ => panic!("SimpleDebug only supports structs"),
    };
    
    let output = quote! {
        impl #impl_generics std::fmt::Debug for #name #ty_generics #where_clause {
            fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
                #debug_body
            }
        }
    };
    
    output.into()
}
```


---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*