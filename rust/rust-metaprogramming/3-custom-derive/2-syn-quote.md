---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-syn-quote"
---

# quote!: Generating Code

The `quote!` macro creates `TokenStream` from code-like syntax:

```rust
use quote::quote;

let name = syn::Ident::new("foo", proc_macro2::Span::call_site());
let value = 42;

let tokens = quote! {
    fn #name() -> i32 {
        #value
    }
};
// Generates: fn foo() -> i32 { 42 }
```

## Interpolation with #

```rust
let name = quote! { MyStruct };
let field = quote! { x };

quote! {
    impl #name {
        fn get_#field(&self) -> i32 {
            self.#field
        }
    }
}
```

## Repetition with #(...)*

```rust
let fields = vec!["a", "b", "c"];

quote! {
    struct MyStruct {
        #( #fields: i32, )*
    }
}
// Generates:
// struct MyStruct {
//     a: i32,
//     b: i32,
//     c: i32,
// }
```

## Complex Repetition

```rust
let names = vec!["x", "y"];
let types = vec![quote!(i32), quote!(String)];

quote! {
    struct Point {
        #( #names: #types, )*
    }
}
// Generates:
// struct Point {
//     x: i32,
//     y: String,
// }
```

## Error Handling

```rust
use syn::Error;
use proc_macro2::Span;

fn validate(input: &DeriveInput) -> Result<(), syn::Error> {
    if some_condition {
        return Err(Error::new(
            Span::call_site(),
            "This macro only works on structs with named fields"
        ));
    }
    Ok(())
}

// In the macro:
match validate(&input) {
    Ok(()) => { /* generate code */ }
    Err(e) => return e.to_compile_error().into(),
}
```

## Code Examples

**Builder pattern derive**

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput, Data, Fields, Error};
use quote::quote;

#[proc_macro_derive(Builder)]
pub fn builder_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    
    match generate_builder(&input) {
        Ok(tokens) => tokens.into(),
        Err(e) => e.to_compile_error().into(),
    }
}

fn generate_builder(input: &DeriveInput) -> Result<proc_macro2::TokenStream, Error> {
    let name = &input.ident;
    let builder_name = syn::Ident::new(
        &format!("{}Builder", name),
        name.span(),
    );
    
    let fields = match &input.data {
        Data::Struct(data) => match &data.fields {
            Fields::Named(f) => &f.named,
            _ => return Err(Error::new_spanned(
                input,
                "Builder requires named fields"
            )),
        },
        _ => return Err(Error::new_spanned(
            input,
            "Builder only works on structs"
        )),
    };
    
    let builder_fields = fields.iter().map(|f| {
        let name = &f.ident;
        let ty = &f.ty;
        quote! { #name: Option<#ty> }
    });
    
    let builder_setters = fields.iter().map(|f| {
        let name = &f.ident;
        let ty = &f.ty;
        quote! {
            pub fn #name(mut self, value: #ty) -> Self {
                self.#name = Some(value);
                self
            }
        }
    });
    
    Ok(quote! {
        pub struct #builder_name {
            #(#builder_fields,)*
        }
        
        impl #builder_name {
            #(#builder_setters)*
        }
        
        impl #name {
            pub fn builder() -> #builder_name {
                #builder_name {
                    #(#fields: None,)*
                }
            }
        }
    })
}
```


---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*