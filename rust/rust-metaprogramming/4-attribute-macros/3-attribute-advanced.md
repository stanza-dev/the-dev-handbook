---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-attribute-advanced"
---

# Advanced Attribute Patterns

## Introduction
Beyond basic function wrapping, attribute macros can transform structs, generate entire modules, and compose with other macros. This lesson covers advanced patterns used in production Rust libraries: conditional code generation, struct transformation, and multi-item output. These patterns are the building blocks of frameworks like Actix-web, Rocket, and Axum.

## Key Concepts
- **Struct transformation**: An attribute macro on a struct can add, remove, or modify fields and generate associated impl blocks.
- **Multi-item output**: A single attribute macro can output multiple items (the modified struct plus impl blocks, plus free functions).
- **Conditional generation**: Using parsed arguments to conditionally include or exclude parts of the generated code.
- **Composability**: Attribute macros can coexist with derive macros and other attributes on the same item.

## Real World Context
Rocket's `#[get("/")]` attribute transforms a function into a route handler with path extraction, request guards, and response conversion. Actix-web's `#[actix_web::main]` rewrites the main function to run inside an Actix runtime. These macros generate far more code than what the user writes, orchestrating complex framework machinery behind a clean annotation.

## Deep Dive

### Transforming a Struct

An attribute macro can modify a struct's fields and generate associated implementations:

```rust
#[proc_macro_attribute]
pub fn entity(args: TokenStream, input: TokenStream) -> TokenStream {
    let input_struct = parse_macro_input!(input as ItemStruct);
    let name = &input_struct.ident;
    let vis = &input_struct.vis;
    let attrs = &input_struct.attrs;

    let existing_fields = match &input_struct.fields {
        Fields::Named(f) => &f.named,
        _ => return Error::new_spanned(name, "expected named fields")
            .to_compile_error().into(),
    };

    // Output: original struct with added 'id' field + impl block
    let output = quote! {
        #(#attrs)*
        #vis struct #name {
            pub id: uuid::Uuid,
            #existing_fields
        }

        impl #name {
            pub fn new(#existing_fields) -> Self {
                Self {
                    id: uuid::Uuid::new_v4(),
                    #(#existing_fields,)*
                }
            }
        }
    };

    output.into()
}
```

The macro adds an `id` field to the struct and generates a `new()` constructor that automatically generates the UUID.

### Conditional Code Generation

Use parsed arguments to control what code gets generated:

```rust
struct MiddlewareArgs {
    auth: bool,
    logging: bool,
}

// In the macro:
let auth_check = if args.auth {
    quote! {
        if !request.is_authenticated() {
            return Err(Error::Unauthorized);
        }
    }
} else {
    quote! {}
};

let log_stmt = if args.logging {
    quote! {
        eprintln!("[REQUEST] {} {}", request.method(), request.path());
    }
} else {
    quote! {}
};

quote! {
    #fn_vis #fn_sig {
        #auth_check
        #log_stmt
        #fn_block
    }
}
```

The generated code only includes auth checking and logging when the user requests them via `#[handler(auth, logging)]`.

### Composing with Derive

Attribute macros run before derive macros, so they can modify a struct that will later be processed by derive:

```rust
#[entity]            // Adds the 'id' field first
#[derive(Debug)]     // Then Debug is derived, including 'id'
struct User {
    name: String,
    email: String,
}
```

The attribute macro adds the `id` field, and then `Debug` is derived on the complete struct including `id`.

### Generating Multiple Items

A single attribute macro can output multiple top-level items:

```rust
let output = quote! {
    // The modified struct
    #vis struct #name { #(#fields,)* }

    // An impl block
    impl #name {
        pub fn new() -> Self { /* ... */ }
    }

    // A free function
    #vis fn #register_fn_name(registry: &mut Registry) {
        registry.register::<#name>();
    }
};
```

The compiler accepts multiple items from a single macro expansion.

## Common Pitfalls
1. **Order of macro application** — Attribute macros run from outermost to innermost, and before derive macros. If your attribute adds fields, derive macros will see the modified struct. If two attribute macros conflict, order matters.
2. **Losing user attributes** — When reconstructing a struct, remember to include `#(#attrs)*` to preserve the user's other attributes (`#[derive(...)]`, `#[cfg(...)]`, etc.).
3. **Generating invalid combinations** — If your macro adds a field to a struct, ensure the field's type is in scope. Users may need to add a dependency or import.

## Best Practices
1. **Document the transformation** — Users should understand what their code becomes after macro expansion. Provide examples in doc comments showing before/after.
2. **Keep generated code readable** — Use `cargo expand` to verify that the generated code is clean. Add comments to generated code with `quote! { // Generated by my_macro }`.
3. **Test edge cases** — Test with empty structs, generic structs, structs with visibility modifiers, and structs with `#[cfg]` attributes.

## Summary
- Attribute macros can transform structs by adding, removing, or modifying fields.
- Conditional generation uses parsed arguments to include/exclude code sections.
- A single macro expansion can output multiple items (struct + impl + functions).
- Attribute macros run before derive macros, enabling composition.
- Always preserve user attributes when reconstructing items.

## Code Examples

**An attribute macro with boolean flag arguments that conditionally generates auth and logging middleware around a function**

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, ItemFn, parse::Parse, parse::ParseStream, Token, Ident};
use quote::quote;

// Parse boolean flags: #[middleware(auth, logging)]
struct MiddlewareArgs {
    auth: bool,
    logging: bool,
}

impl Parse for MiddlewareArgs {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let mut auth = false;
        let mut logging = false;

        while !input.is_empty() {
            let flag: Ident = input.parse()?;
            match flag.to_string().as_str() {
                "auth" => auth = true,
                "logging" => logging = true,
                other => return Err(syn::Error::new(
                    flag.span(),
                    format!("unknown flag '{}', expected 'auth' or 'logging'", other)
                )),
            }
            if !input.is_empty() {
                input.parse::<Token![,]>()?;
            }
        }
        Ok(MiddlewareArgs { auth, logging })
    }
}

#[proc_macro_attribute]
pub fn middleware(args: TokenStream, input: TokenStream) -> TokenStream {
    let args = parse_macro_input!(args as MiddlewareArgs);
    let input_fn = parse_macro_input!(input as ItemFn);
    let fn_name = &input_fn.sig.ident;
    let fn_vis = &input_fn.vis;
    let fn_sig = &input_fn.sig;
    let fn_block = &input_fn.block;

    let auth_check = if args.auth {
        quote! { eprintln!("[AUTH] Checking authentication for {}", stringify!(#fn_name)); }
    } else {
        quote! {}
    };

    let log_entry = if args.logging {
        quote! { eprintln!("[LOG] Entering {}", stringify!(#fn_name)); }
    } else {
        quote! {}
    };

    quote! {
        #fn_vis #fn_sig {
            #auth_check
            #log_entry
            #fn_block
        }
    }.into()
}

// Usage:
// #[middleware(auth, logging)]
// fn handle_request() -> Response { ... }
```


## Resources

- [Attribute Macros - The Rust Reference](https://doc.rust-lang.org/reference/procedural-macros.html#attribute-macros) — Official reference for attribute macro syntax, semantics, and usage patterns
- [Procedural Macros Workshop](https://github.com/dtolnay/proc-macro-workshop) — David Tolnay's hands-on workshop with exercises for building derive, attribute, and function-like proc macros

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*