---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-quote-basics"
---

# Code Generation with quote

## Introduction
The `quote` crate is the code generation half of the proc macro toolkit. Where `syn` parses tokens into an AST, `quote` converts your generated AST back into tokens. Its `quote!` macro lets you write Rust code templates with interpolation, producing `TokenStream` values ready for the compiler. Together with `syn`, it forms a complete parse-transform-emit pipeline.

## Key Concepts
- **`quote!` macro**: Creates a `proc_macro2::TokenStream` from a Rust code template with `#variable` interpolation.
- **`#variable` interpolation**: Inside `quote!`, prefixing a variable with `#` inserts its value into the generated token stream.
- **`#( #var )*` repetition**: Iterates over a collection, emitting the template once for each element — analogous to `$( )` in declarative macros.
- **`quote_spanned!`**: Like `quote!` but attaches a specific source location to the generated tokens, improving error messages.

## Real World Context
Every derive macro that generates trait implementations uses `quote!`. When serde generates `impl Serialize for MyStruct { ... }`, it builds that code with `quote!`. When you see a derive macro produce a compile error pointing at a specific field, that is `quote_spanned!` at work. Understanding `quote!` interpolation and repetition is essential for producing correct, readable generated code.

## Deep Dive

### Basic Interpolation

The `#` prefix inside `quote!` inserts the value of a Rust variable:

```rust
use quote::quote;
use syn::Ident;
use proc_macro2::Span;

let function_name = Ident::new("compute_total", Span::call_site());
let default_value = 42i32;

let tokens = quote! {
    fn #function_name() -> i32 {
        #default_value
    }
};
// Generates: fn compute_total() -> i32 { 42 }
```

The variable `function_name` is an `Ident`, so it becomes a Rust identifier in the output. The integer `default_value` becomes a literal. The `quote!` macro automatically handles the conversion based on the variable's type.

### Repetition

To generate repeated code from a collection, use `#( )` with `*` or `,` separator:

```rust
let field_names = vec!["name", "email", "age"];

let tokens = quote! {
    fn describe() {
        #( println!("Field: {}", #field_names); )*
    }
};
// Generates:
// fn describe() {
//     println!("Field: {}", "name");
//     println!("Field: {}", "email");
//     println!("Field: {}", "age");
// }
```

For comma-separated lists, add a comma before the `*`:

```rust
let values = vec![1i32, 2, 3];
let tokens = quote! {
    let items = vec![#( #values ),*];
};
// Generates: let items = vec![1, 2, 3];
```

### Multiple Variables in Repetition

When two iterables have the same length, you can zip them in a single repetition block:

```rust
let names = vec!["width", "height"];
let types = vec![quote!(f64), quote!(f64)];

let tokens = quote! {
    struct Rectangle {
        #( #names: #types, )*
    }
};
// Generates:
// struct Rectangle {
//     width: f64,
//     height: f64,
// }
```

Both `names` and `types` advance together on each iteration.

### Error Handling with Spans

Use `syn::Error` to produce compile errors that point to the right location in the user's source code:

```rust
use syn::Error;
use proc_macro2::Span;

fn validate(input: &DeriveInput) -> Result<(), syn::Error> {
    match &input.data {
        Data::Struct(_) => Ok(()),
        _ => Err(Error::new_spanned(
            &input.ident,
            "This macro only supports structs with named fields"
        )),
    }
}

// In the macro entry point:
match validate(&input) {
    Ok(()) => { /* generate code with quote! */ }
    Err(err) => return err.to_compile_error().into(),
}
```

The `Error::new_spanned` method attaches the error to the token that caused the problem, so the compiler underlines the right part of the user's code.

## Common Pitfalls
1. **Forgetting `into()` when returning** — `quote!` returns `proc_macro2::TokenStream`, but proc macro functions return `proc_macro::TokenStream`. Call `.into()` to convert.
2. **Mismatched repetition lengths** — If two variables in a `#( #a #b )*` block have different lengths, you get a compile-time panic. Ensure iterables are the same length.
3. **Using `panic!` instead of `syn::Error`** — A panic in a proc macro produces an incomprehensible error. Always use `syn::Error` with spans for user-facing errors.

## Best Practices
1. **Use `quote_spanned!` for field-level errors** — Attach the span of the specific field or expression that caused an issue, not the whole struct.
2. **Extract generation into helper functions** — Keep the `#[proc_macro_derive]` function thin. Put the logic in a function returning `Result<proc_macro2::TokenStream, syn::Error>` for testability.
3. **Inspect output with `cargo expand`** — Run `cargo expand` to see the actual code your macro generates. This is essential for debugging subtle issues.

## Summary
- `quote!` generates TokenStream from Rust code templates with `#variable` interpolation.
- Repetition with `#( )*` iterates over collections in the generated code.
- Multiple variables in the same repetition block advance in lockstep.
- Use `syn::Error` with spans for clear, well-located compile errors.
- Call `.into()` to convert `proc_macro2::TokenStream` to `proc_macro::TokenStream`.

## Code Examples

**A complete derive macro using quote! repetition and syn::Error for clean error handling — generates a describe() method listing all fields and their types**

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput, Data, Fields, Error};
use quote::quote;

#[proc_macro_derive(Describe)]
pub fn describe_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    match generate_describe(&input) {
        Ok(tokens) => tokens.into(),
        Err(err) => err.to_compile_error().into(),
    }
}

fn generate_describe(input: &DeriveInput) -> Result<proc_macro2::TokenStream, Error> {
    let name = &input.ident;
    let name_str = name.to_string();

    let fields = match &input.data {
        Data::Struct(data) => match &data.fields {
            Fields::Named(f) => &f.named,
            _ => return Err(Error::new_spanned(name, "Describe requires named fields")),
        },
        _ => return Err(Error::new_spanned(name, "Describe only works on structs")),
    };

    let field_descriptions = fields.iter().map(|f| {
        let field_name = &f.ident;
        let field_type = &f.ty;
        quote! {
            println!("  {}: {}", stringify!(#field_name), stringify!(#field_type));
        }
    });

    Ok(quote! {
        impl #name {
            pub fn describe() {
                println!("Struct: {}", #name_str);
                #( #field_descriptions )*
            }
        }
    })
}
```


## Resources

- [quote crate documentation](https://docs.rs/quote/1/quote/) — API reference for the quote crate including the quote! macro, repetition syntax, and ToTokens trait

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*