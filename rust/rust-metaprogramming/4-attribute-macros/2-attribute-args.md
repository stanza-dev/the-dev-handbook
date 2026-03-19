---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-attribute-args"
---

# Parsing Attribute Arguments

## Introduction
Attribute macros become truly powerful when they accept arguments that configure their behavior. Arguments like `#[route(method = "GET", path = "/users")]` or `#[cache(ttl = 300)]` let users customize what the macro generates. Parsing these arguments correctly, with helpful error messages for invalid input, is a core proc macro skill. In syn 2.0, argument parsing uses the `parse_nested_meta` callback API.

## Key Concepts
- **Attribute arguments**: The tokens inside the parentheses of an attribute: `#[my_attr(these tokens)]`.
- **`parse_nested_meta()`**: The syn 2.0 method on `Attribute` and `MetaList` for parsing comma-separated key-value arguments with a callback.
- **Custom `Parse` implementation**: For complex argument structures, implement `syn::parse::Parse` to parse the entire argument stream into a structured type.
- **`ParseStream`**: The buffered token stream type used by syn's parsing infrastructure, providing `parse()`, `peek()`, and `lookahead1()` methods.

## Real World Context
Every attribute macro with configurable behavior needs argument parsing. Tokio's `#[tokio::main(flavor = "multi_thread", worker_threads = 4)]` parses two named arguments. Actix-web's `#[get("/users/{id}")]` parses a path string. Serde's field attributes parse options like `rename`, `default`, and `skip`. Clean argument parsing with good error messages is what separates a polished macro from a frustrating one.

## Deep Dive

### Simple Argument Parsing

For macros with a small number of known arguments, parse the args `TokenStream` directly:

```rust
use syn::{parse::Parse, parse::ParseStream, Token, LitStr, LitInt, Ident};

struct CacheArgs {
    ttl_seconds: u64,
    key_prefix: Option<String>,
}

impl Parse for CacheArgs {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let mut ttl_seconds = 60; // default
        let mut key_prefix = None;

        while !input.is_empty() {
            let key: Ident = input.parse()?;
            input.parse::<Token![=]>()?;

            if key == "ttl" {
                let value: LitInt = input.parse()?;
                ttl_seconds = value.base10_parse()?;
            } else if key == "prefix" {
                let value: LitStr = input.parse()?;
                key_prefix = Some(value.value());
            } else {
                return Err(syn::Error::new(
                    key.span(),
                    format!("unknown argument '{}', expected 'ttl' or 'prefix'", key)
                ));
            }

            if !input.is_empty() {
                input.parse::<Token![,]>()?;
            }
        }

        Ok(CacheArgs { ttl_seconds, key_prefix })
    }
}
```

This parser handles `#[cache(ttl = 300, prefix = "user")]` and produces a clear error for unrecognized keys.

### Using Custom Parse in the Macro

Use `parse_macro_input!` to convert the args stream into your custom type:

```rust
#[proc_macro_attribute]
pub fn cache(args: TokenStream, input: TokenStream) -> TokenStream {
    let args = parse_macro_input!(args as CacheArgs);
    let input_fn = parse_macro_input!(input as ItemFn);

    let ttl = args.ttl_seconds;
    let fn_name = &input_fn.sig.ident;
    let fn_vis = &input_fn.vis;
    let fn_sig = &input_fn.sig;
    let fn_block = &input_fn.block;

    let output = quote! {
        #fn_vis #fn_sig {
            static CACHE_TTL: u64 = #ttl;
            eprintln!("{}: cache TTL is {}s", stringify!(#fn_name), CACHE_TTL);
            #fn_block
        }
    };

    output.into()
}
```

The macro cleanly separates parsing from code generation.

### Parsing with parse_nested_meta (for derive helper attributes)

When parsing attributes on fields or types (as in derive macros), use `parse_nested_meta()`:

```rust
use syn::Attribute;

fn parse_config(attr: &Attribute) -> syn::Result<(bool, Option<String>)> {
    let mut skip = false;
    let mut rename = None;

    attr.parse_nested_meta(|meta| {
        if meta.path.is_ident("skip") {
            skip = true;
            Ok(())
        } else if meta.path.is_ident("rename") {
            let value = meta.value()?;
            let lit: syn::LitStr = value.parse()?;
            rename = Some(lit.value());
            Ok(())
        } else {
            Err(meta.error("expected `skip` or `rename`"))
        }
    })?;

    Ok((skip, rename))
}
```

This handles `#[my_attr(skip)]` and `#[my_attr(rename = "other_name")]` with proper error reporting.

### Boolean Flags vs Key-Value Pairs

Arguments come in two common forms:

```rust
// Boolean flag: just the name, no value
#[my_attr(verbose, async)]

// Key-value pair: name = value
#[my_attr(timeout = 30, name = "worker")]

// Mixed:
#[my_attr(verbose, timeout = 30)]
```

In `parse_nested_meta`, check `meta.input.peek(Token![=])` to distinguish flags from key-value pairs:

```rust
attr.parse_nested_meta(|meta| {
    if meta.path.is_ident("verbose") {
        // Flag: no value expected
        verbose = true;
        Ok(())
    } else if meta.path.is_ident("timeout") {
        // Key-value: parse the = and the value
        let value = meta.value()?;
        let lit: syn::LitInt = value.parse()?;
        timeout = lit.base10_parse()?;
        Ok(())
    } else {
        Err(meta.error("unrecognized attribute"))
    }
})?;
```

## Common Pitfalls
1. **Not consuming commas between arguments** — When implementing `Parse` manually, you must consume the `,` token between arguments or the parser gets stuck.
2. **Silently ignoring unknown arguments** — Always error on unrecognized keys. Users who misspell an argument name deserve a clear error, not silent incorrect behavior.
3. **Using syn 1.x `AttributeArgs`** — The `AttributeArgs` type alias and `NestedMeta` enum were removed in syn 2.0. Use `parse_nested_meta()` or custom `Parse` implementations instead.

## Best Practices
1. **Provide defaults** — Most arguments should have reasonable defaults. Only error on genuinely required arguments.
2. **Validate combinations** — Check for contradictory arguments (e.g., `#[my_attr(sync, async)]`) and produce a clear error.
3. **Use `Span` for errors** — Attach errors to the specific argument that caused the problem, not the entire attribute.

## Summary
- Attribute arguments are parsed from the first `TokenStream` parameter.
- Implement `syn::parse::Parse` for complex argument structures.
- Use `parse_nested_meta()` for derive helper attributes in syn 2.0.
- Always reject unknown arguments with clear, span-attached errors.
- `AttributeArgs` and `NestedMeta` are removed in syn 2.0.

## Code Examples

**A complete attribute macro with custom argument parsing using syn 2.0 APIs — parses key=value arguments and wraps the function with configurable logging**

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, ItemFn, parse::Parse, parse::ParseStream, Token, Ident, LitStr};
use quote::quote;

// Parse arguments like: #[log_call(level = "debug")]
struct LogArgs {
    level: String,
}

impl Parse for LogArgs {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let mut level = "info".to_string();

        while !input.is_empty() {
            let key: Ident = input.parse()?;
            input.parse::<Token![=]>()?;

            if key == "level" {
                let value: LitStr = input.parse()?;
                level = value.value();
            } else {
                return Err(syn::Error::new(key.span(),
                    format!("unknown argument '{}', expected 'level'", key)));
            }

            if !input.is_empty() {
                input.parse::<Token![,]>()?;
            }
        }
        Ok(LogArgs { level })
    }
}

#[proc_macro_attribute]
pub fn log_call(args: TokenStream, input: TokenStream) -> TokenStream {
    let args = parse_macro_input!(args as LogArgs);
    let input_fn = parse_macro_input!(input as ItemFn);

    let level = &args.level;
    let fn_name = &input_fn.sig.ident;
    let fn_vis = &input_fn.vis;
    let fn_sig = &input_fn.sig;
    let fn_block = &input_fn.block;
    let fn_attrs = &input_fn.attrs;

    let output = quote! {
        #(#fn_attrs)*
        #fn_vis #fn_sig {
            eprintln!("[{}] Entering {}", #level, stringify!(#fn_name));
            let __result = { #fn_block };
            eprintln!("[{}] Leaving {}", #level, stringify!(#fn_name));
            __result
        }
    };

    output.into()
}

// Usage:
// #[log_call(level = "debug")]
// fn process_order(order_id: u64) -> Result<(), Error> { ... }
```


## Resources

- [syn parse module documentation](https://docs.rs/syn/2/syn/parse/index.html) — API reference for syn's parsing infrastructure including Parse trait, ParseStream, and ParseBuffer

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*