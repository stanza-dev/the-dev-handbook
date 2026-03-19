---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-derive-implementation"
---

# Creating a Derive Macro

## Introduction
Custom derive macros are the most common type of procedural macro. They let you write `#[derive(MyTrait)]` on a struct or enum and have the compiler generate a trait implementation automatically. This section walks through creating a derive macro step by step, from the simplest case to handling generics correctly.

## Key Concepts
- **`#[proc_macro_derive(Name)]`**: The attribute that marks a function as a custom derive macro. The `Name` becomes the identifier used in `#[derive(Name)]`.
- **`split_for_impl()`**: A method on `syn::Generics` that splits generic parameters into the three parts needed for an `impl` block: `impl<T> Name<T> where T: ...`.
- **Additive only**: Derive macros can only *add* new items (impl blocks, functions). They cannot modify the original struct or enum.

## Real World Context
Almost every Rust project uses custom derive macros. `#[derive(Debug, Clone, Serialize, Deserialize)]` is ubiquitous. Understanding how to create your own means you can eliminate boilerplate trait implementations specific to your domain — for example, deriving a `Validate` trait that checks struct fields against business rules.

## Deep Dive

### Step 1: Basic Implementation

Start with the simplest possible derive macro that generates a method:

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput};
use quote::quote;

#[proc_macro_derive(Hello)]
pub fn hello_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    let name_str = name.to_string();

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

This generates an `impl` block with a `hello()` method. The `#name` interpolation inserts the struct's name as an identifier, and `#name_str` inserts it as a string literal.

### Step 2: Handle Generics

The basic version breaks for generic types like `Person<T>`. Use `split_for_impl()` to handle generics correctly:

```rust
#[proc_macro_derive(Hello)]
pub fn hello_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;

    // Split generics into the three parts of an impl block
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

The `split_for_impl()` method returns three parts: `impl_generics` goes after `impl`, `ty_generics` goes after the type name, and `where_clause` goes at the end. For `struct Person<T: Display>`, these expand to `<T: Display>`, `<T>`, and `` respectively.

### Step 3: Use It

```rust
use my_derive::Hello;

#[derive(Hello)]
struct Person<T> {
    name: String,
    data: T,
}

fn main() {
    let person = Person { name: "Alice".into(), data: 42 };
    person.hello();  // Output: "Hello from Person!"
}
```

The derive macro correctly generates `impl<T> Person<T> { ... }`, handling the generic parameter transparently.

### Step 3: Process Fields

Most real derive macros need to iterate over the struct's fields to generate field-specific code:

```rust
let field_prints = match &input.data {
    Data::Struct(data) => match &data.fields {
        Fields::Named(fields) => {
            fields.named.iter().map(|f| {
                let field_name = &f.ident;
                quote! {
                    println!("  {}: {:?}", stringify!(#field_name), &self.#field_name);
                }
            }).collect::<Vec<_>>()
        }
        _ => return Error::new_spanned(name, "expected named fields")
            .to_compile_error().into(),
    },
    _ => return Error::new_spanned(name, "Hello only works on structs")
        .to_compile_error().into(),
};
```

This maps each named field to a `println!` statement that prints the field's name and value using Debug formatting.

## Common Pitfalls
1. **Ignoring `split_for_impl()`** — Writing `impl #name { ... }` without generics works for concrete types but fails for any generic struct. Always use `split_for_impl()` even if you think generics are unlikely.
2. **Returning the wrong tokens** — Derive macros must return *new* code to add, not a modified version of the input. The compiler appends your output after the original item.
3. **Missing trait bounds** — If your generated code calls `.clone()` on a field, you need to add a `T: Clone` bound. Use `where_clause` from `split_for_impl()` and extend it with additional bounds.

## Best Practices
1. **Always handle generics** — Use `split_for_impl()` from the start, even in tutorials. It costs nothing for non-generic types and prevents breakage later.
2. **Generate trait impls, not inherent impls** — Prefer `impl MyTrait for #name` over `impl #name`. Trait impls are namespaced, avoiding conflicts with user-defined methods.
3. **Write integration tests** — Create a `tests/` directory in your derive crate with structs that exercise named fields, tuple structs, generics, and lifetimes.

## Summary
- Custom derive macros add code alongside the annotated type using `#[proc_macro_derive(Name)]`.
- `split_for_impl()` splits generics into the three parts needed for an impl block.
- Derive macros are additive — they cannot modify the original item.
- Always handle generics and use `syn::Error` for unsupported cases.
- Prefer trait implementations over inherent impl blocks.

## Code Examples

**A complete SimpleDebug derive macro handling named fields, tuple structs, and unit structs with proper generic support**

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput, Data, Fields, Error};
use quote::quote;

// Generates a Debug-like implementation
#[proc_macro_derive(SimpleDebug)]
pub fn simple_debug_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    let (impl_generics, ty_generics, where_clause) =
        input.generics.split_for_impl();

    let debug_body = match &input.data {
        Data::Struct(data) => match &data.fields {
            Fields::Named(fields) => {
                let field_prints = fields.named.iter().map(|f| {
                    let field_name = &f.ident;
                    quote! { .field(stringify!(#field_name), &self.#field_name) }
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
            Fields::Unit => quote! { f.write_str(stringify!(#name)) },
        },
        _ => return Error::new_spanned(name, "SimpleDebug only supports structs")
            .to_compile_error().into(),
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


## Resources

- [Derive Macros - The Rust Reference](https://doc.rust-lang.org/reference/procedural-macros.html#derive-macros) — Official reference for derive macro rules, helper attributes, and resolution

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*