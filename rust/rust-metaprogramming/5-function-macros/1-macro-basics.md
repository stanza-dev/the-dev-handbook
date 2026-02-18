---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-function-macro-basics"
---

# Function-Like Procedural Macros

Called like `macro!()` but with full proc-macro power.

```rust
#[proc_macro]
pub fn my_macro(input: TokenStream) -> TokenStream {
    // Parse input, generate output
    input
}
```

## Example: SQL Query Builder

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, LitStr};

#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    let query = parse_macro_input!(input as LitStr);
    let query_str = query.value();
    
    // Validate SQL at compile time!
    if !query_str.to_uppercase().starts_with("SELECT") {
        return syn::Error::new(
            query.span(),
            "Only SELECT queries allowed"
        ).to_compile_error().into();
    }
    
    quote! {
        Query::new(#query_str)
    }.into()
}

// Usage:
let q = sql!("SELECT * FROM users");  // OK
// let q = sql!("DROP TABLE users");  // Compile error!
```

## Example: JSON Literal

```rust
#[proc_macro]
pub fn json(input: TokenStream) -> TokenStream {
    // Parse JSON-like syntax
    // { "key": value, ... }
    
    quote! {
        serde_json::json!(#input)
    }.into()
}

// Usage:
let data = json!({
    "name": "Alice",
    "age": 30
});
```

## Code Examples

**HTML builder macro**

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::parse::{Parse, ParseStream};
use syn::{Token, Ident, LitStr, punctuated::Punctuated};

// HTML macro: html!(div class="foo" { "Hello" })
struct HtmlNode {
    tag: Ident,
    attrs: Vec<(Ident, LitStr)>,
    content: Option<LitStr>,
}

impl Parse for HtmlNode {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let tag: Ident = input.parse()?;
        
        let mut attrs = Vec::new();
        while input.peek(Ident) && !input.peek(syn::token::Brace) {
            let name: Ident = input.parse()?;
            input.parse::<Token![=]>()?;
            let value: LitStr = input.parse()?;
            attrs.push((name, value));
        }
        
        let content = if input.peek(syn::token::Brace) {
            let content_braced;
            syn::braced!(content_braced in input);
            Some(content_braced.parse()?)
        } else {
            None
        };
        
        Ok(HtmlNode { tag, attrs, content })
    }
}

#[proc_macro]
pub fn html(input: TokenStream) -> TokenStream {
    let node = syn::parse_macro_input!(input as HtmlNode);
    
    let tag = node.tag.to_string();
    let attrs: Vec<_> = node.attrs.iter().map(|(k, v)| {
        let key = k.to_string();
        let val = v.value();
        format!(" {}=\"{}\"", key, val)
    }).collect();
    let attr_str = attrs.join("");
    
    let content = node.content.map(|c| c.value()).unwrap_or_default();
    
    let result = format!("<{0}{1}>{2}</{0}>", tag, attr_str, content);
    
    quote! { #result }.into()
}

// Usage:
let s = html!(div class="container" { "Hello World" });
// s = "<div class=\"container\">Hello World</div>"
```


---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*