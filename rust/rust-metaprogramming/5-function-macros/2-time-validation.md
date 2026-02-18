---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-compile-time-validation"
---

# The Power of Compile-Time Checks

Procedural macros can validate input at compile time:

## Regex Validation

```rust
#[proc_macro]
pub fn regex(input: TokenStream) -> TokenStream {
    let lit = parse_macro_input!(input as LitStr);
    let pattern = lit.value();
    
    // Validate regex at compile time
    if let Err(e) = regex::Regex::new(&pattern) {
        return syn::Error::new(
            lit.span(),
            format!("Invalid regex: {}", e)
        ).to_compile_error().into();
    }
    
    quote! {
        regex::Regex::new(#pattern).unwrap()
    }.into()
}

// Usage:
let re = regex!(r"\d{3}-\d{4}");  // Validated!
// let bad = regex!(r"[invalid");  // Compile error!
```

## Format String Validation

```rust
#[proc_macro]
pub fn format_checked(input: TokenStream) -> TokenStream {
    // Parse format string and arguments
    // Validate placeholder count matches arguments
    // Return compile error if mismatch
    todo!()
}
```

## Type-Safe Routing

```rust
#[proc_macro]
pub fn route(input: TokenStream) -> TokenStream {
    // Parse: "/users/:id/posts/:post_id"
    // Generate: struct with id: String, post_id: String
    // Validate path segments
    todo!()
}

// Usage:
let r = route!("/users/:id");
// r.id is available
// r.nonexistent causes compile error
```

## Benefits

1. **No Runtime Panics**: Errors caught at compile time
2. **Better Errors**: Custom error messages with spans
3. **Zero Runtime Cost**: Validation happens once, at compile time
4. **IDE Support**: Errors show in your editor

## Code Examples

**Compile-time validation examples**

```rust
// Environment variable with compile-time existence check
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, LitStr};

#[proc_macro]
pub fn env_or_fail(input: TokenStream) -> TokenStream {
    let var_name = parse_macro_input!(input as LitStr);
    let name = var_name.value();
    
    // Check at compile time that env var exists
    // (Useful for required config during build)
    if std::env::var(&name).is_err() {
        // Use cfg to make this optional
        #[cfg(not(feature = "skip-env-check"))]
        return syn::Error::new(
            var_name.span(),
            format!("Required environment variable '{}' is not set", name)
        ).to_compile_error().into();
    }
    
    quote! {
        std::env::var(#name).expect(
            concat!("Environment variable ", #name, " must be set")
        )
    }.into()
}

// Usage:
// DATABASE_URL must be set when compiling:
// let url = env_or_fail!("DATABASE_URL");

// Another example: validate JSON schema at compile time
#[proc_macro]
pub fn json_schema(input: TokenStream) -> TokenStream {
    let schema_str = parse_macro_input!(input as LitStr);
    
    // Parse and validate JSON at compile time
    let schema: serde_json::Value = match serde_json::from_str(&schema_str.value()) {
        Ok(v) => v,
        Err(e) => return syn::Error::new(
            schema_str.span(),
            format!("Invalid JSON: {}", e)
        ).to_compile_error().into(),
    };
    
    // Could also validate it's a valid JSON Schema
    
    quote! {
        serde_json::from_str::<serde_json::Value>(#schema_str).unwrap()
    }.into()
}
```


---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*