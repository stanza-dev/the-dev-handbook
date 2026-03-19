---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-function-macro-basics"
---

# Creating Function-Like Macros

## Introduction
Function-like procedural macros are invoked with the `my_macro!(...)` syntax, just like declarative macros, but with the full power of Rust's type system, standard library, and external crates available at compile time. They are ideal for building domain-specific languages (DSLs), validating inputs at compile time, and generating code from custom syntax that `macro_rules!` cannot parse.

## Key Concepts
- **`#[proc_macro]`**: The attribute that marks a function as a function-like proc macro. It receives one `TokenStream` parameter.
- **Custom parsing**: Function-like macros can parse any syntax — not just Rust syntax. By implementing `syn::parse::Parse`, you define your own mini-language.
- **Compile-time validation**: Because the macro runs during compilation, it can validate inputs (SQL, regex, JSON) and reject invalid ones with compile errors.
- **DSL (Domain-Specific Language)**: A small, focused language embedded within Rust through a macro, offering syntax tailored to a specific problem domain.

## Real World Context
SQLx's `sqlx::query!("SELECT * FROM users WHERE id = $1")` validates SQL syntax and types at compile time. The `html!` macro in Yew provides JSX-like syntax for Rust web applications. The `lazy_static!` macro (now largely superseded by `std::sync::LazyLock`) used function-like macro syntax. These macros catch entire classes of errors at compile time that would otherwise surface as runtime panics.

## Deep Dive

### Basic Structure

A function-like proc macro receives one `TokenStream` and returns another:

```rust
use proc_macro::TokenStream;

#[proc_macro]
pub fn my_macro(input: TokenStream) -> TokenStream {
    // Parse input tokens
    // Validate
    // Generate output tokens
    input // simplest case: return input unchanged
}
```

The input contains everything between the delimiters: `my_macro!(these tokens)`.

### Example: Compile-Time SQL Validation

Here is a macro that validates SQL query strings at compile time:

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, LitStr};
use quote::quote;

#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    let query = parse_macro_input!(input as LitStr);
    let query_str = query.value();

    // Validate: only allow SELECT queries
    if !query_str.trim().to_uppercase().starts_with("SELECT") {
        return syn::Error::new(
            query.span(),
            "Only SELECT queries are allowed. Use execute!() for mutations."
        ).to_compile_error().into();
    }

    // Validate: check for common SQL injection patterns
    if query_str.contains("--") || query_str.contains(";") {
        return syn::Error::new(
            query.span(),
            "Query contains potentially dangerous characters"
        ).to_compile_error().into();
    }

    quote! {
        Query::new(#query_str)
    }.into()
}
```

Usage demonstrates compile-time safety:

```rust
let q = sql!("SELECT name, email FROM users WHERE active = true"); // Compiles
// let q = sql!("DROP TABLE users"); // Compile error: Only SELECT queries allowed
```

The error appears in the IDE and at compile time, long before the code reaches a database.

### Custom Syntax Parsing

Function-like macros can define entirely custom syntax by implementing the `Parse` trait:

```rust
use syn::parse::{Parse, ParseStream};
use syn::{Token, Ident, LitStr, punctuated::Punctuated};

// Custom syntax: key_value!(name => "Alice", age => "30")
struct KeyValuePairs {
    pairs: Vec<(Ident, LitStr)>,
}

impl Parse for KeyValuePairs {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let mut pairs = Vec::new();

        while !input.is_empty() {
            let key: Ident = input.parse()?;
            input.parse::<Token![=>]>()?;
            let value: LitStr = input.parse()?;
            pairs.push((key, value));

            if !input.is_empty() {
                input.parse::<Token![,]>()?;
            }
        }

        Ok(KeyValuePairs { pairs })
    }
}
```

This parser handles the custom `key => "value"` syntax, which is not valid Rust but is valid within our macro.

### Using the Custom Parser

```rust
#[proc_macro]
pub fn key_value(input: TokenStream) -> TokenStream {
    let kv = parse_macro_input!(input as KeyValuePairs);

    let inserts = kv.pairs.iter().map(|(key, value)| {
        let key_str = key.to_string();
        quote! {
            map.insert(#key_str.to_string(), #value.to_string());
        }
    });

    quote! {
        {
            let mut map = std::collections::HashMap::new();
            #(#inserts)*
            map
        }
    }.into()
}

// Usage:
// let config = key_value!(host => "localhost", port => "8080");
```

## Common Pitfalls
1. **Trying to parse non-Rust syntax with `macro_rules!`** — Declarative macros can only parse valid Rust token patterns. For custom syntax like `key => value`, you need a proc macro with a custom `Parse` implementation.
2. **Forgetting that input is tokens, not strings** — The input `TokenStream` is already tokenized. You cannot do raw string manipulation on it. Parse it into structured types first.
3. **Heavy compile-time dependencies** — If your proc macro depends on a large crate (like a full SQL parser), it increases compile times for every crate that uses your macro.

## Best Practices
1. **Keep the macro crate lightweight** — Only include dependencies needed for parsing and code generation in the proc macro crate. Heavy validation logic can go in a separate utility crate.
2. **Provide clear error messages** — Use `syn::Error::new(span, message)` with the span of the problematic token so the error points to the right place.
3. **Document your DSL syntax** — Custom syntax is invisible to rustfmt and rust-analyzer. Add thorough doc comments and examples showing valid invocations.

## Summary
- Function-like proc macros use `#[proc_macro]` and receive one `TokenStream`.
- They can parse custom syntax beyond what `macro_rules!` supports.
- Compile-time validation catches errors before runtime.
- Implement `syn::parse::Parse` to define custom parsers.
- Keep proc macro crate dependencies lightweight for fast compilation.

## Code Examples

**A complete function-like macro implementing an HTML DSL with custom syntax parsing for tags, attributes, and content**

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, parse::{Parse, ParseStream}, Token, Ident, LitStr};
use quote::quote;

// Custom DSL: html!(div class="container" { "Hello World" })
struct HtmlNode {
    tag: Ident,
    attrs: Vec<(Ident, LitStr)>,
    content: Option<LitStr>,
}

impl Parse for HtmlNode {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let tag: Ident = input.parse()?;

        let mut attrs = Vec::new();
        // Parse attributes until we hit a brace or end of input
        while !input.is_empty() && !input.peek(syn::token::Brace) {
            let name: Ident = input.parse()?;
            input.parse::<Token![=]>()?;
            let value: LitStr = input.parse()?;
            attrs.push((name, value));
        }

        // Parse optional content in braces
        let content = if input.peek(syn::token::Brace) {
            let braced;
            syn::braced!(braced in input);
            Some(braced.parse()?)
        } else {
            None
        };

        Ok(HtmlNode { tag, attrs, content })
    }
}

#[proc_macro]
pub fn html(input: TokenStream) -> TokenStream {
    let node = parse_macro_input!(input as HtmlNode);

    let tag = node.tag.to_string();
    let attr_parts: Vec<_> = node.attrs.iter().map(|(k, v)| {
        let key = k.to_string();
        let val = v.value();
        format!(" {}=\"{}\"", key, val)
    }).collect();
    let attrs_str = attr_parts.join("");
    let content = node.content.map(|c| c.value()).unwrap_or_default();
    let result = format!("<{0}{1}>{2}</{0}>", tag, attrs_str, content);

    quote! { #result }.into()
}

// Usage:
// let markup = html!(div class="greeting" { "Hello World" });
// markup == "<div class=\"greeting\">Hello World</div>"
```


## Resources

- [Function-like procedural macros - The Rust Reference](https://doc.rust-lang.org/reference/procedural-macros.html#function-like-procedural-macros) — Official specification for function-like proc macros including invocation rules and token handling

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*