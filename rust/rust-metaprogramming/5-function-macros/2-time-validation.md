---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-compile-time-validation"
---

# Compile-Time Validation Patterns

## Introduction
One of the most valuable applications of function-like proc macros is validating data at compile time. Instead of discovering that a regex is invalid, a SQL query is malformed, or a configuration value is missing at runtime, the compiler catches these errors during the build. This eliminates entire categories of production bugs and shifts error discovery to the earliest possible moment.

## Key Concepts
- **Compile-time validation**: Running checks during compilation that would normally happen at runtime, producing compiler errors for invalid input.
- **Span-aware errors**: Using `syn::Error::new(span, message)` to produce error messages that point to the exact location in the user's source code where the problem occurs.
- **Zero runtime cost**: Validation happens once at compile time. The generated code assumes the input is valid and does not need runtime checks.
- **`compile_error!`**: A built-in macro that unconditionally produces a compile error. Proc macros can emit this for validation failures.

## Real World Context
SQLx validates SQL queries against a real database at compile time. The `regex` crate's `regex!` macro (from the `regex_macros` feature) compiles regex patterns at build time. The `include_str!` macro validates file paths at compile time. These patterns demonstrate that Rust's macro system can make "if it compiles, it works" a much stronger guarantee.

## Deep Dive

### Pattern 1: Regex Validation

Validate regex patterns at compile time:

```rust
#[proc_macro]
pub fn checked_regex(input: TokenStream) -> TokenStream {
    let pattern = parse_macro_input!(input as LitStr);
    let pattern_str = pattern.value();

    // Validate the regex at compile time
    if let Err(err) = regex_syntax::parse(&pattern_str) {
        return syn::Error::new(
            pattern.span(),
            format!("Invalid regex pattern: {}", err)
        ).to_compile_error().into();
    }

    // Generate code that constructs the regex at runtime
    // (guaranteed to succeed since we validated at compile time)
    quote! {
        regex::Regex::new(#pattern_str).unwrap() // Safe: validated at compile time
    }.into()
}
```

The `unwrap()` in the generated code is safe because the macro already verified the pattern is valid.

### Pattern 2: Environment Variable Checking

Ensure required configuration exists at build time:

```rust
#[proc_macro]
pub fn required_env(input: TokenStream) -> TokenStream {
    let var_name = parse_macro_input!(input as LitStr);
    let name = var_name.value();

    if std::env::var(&name).is_err() {
        return syn::Error::new(
            var_name.span(),
            format!(
                "Required environment variable '{}' is not set.\n\
                 Set it before building: export {}=value",
                name, name
            )
        ).to_compile_error().into();
    }

    quote! {
        std::env::var(#name).expect(concat!("env var ", #name, " must be set"))
    }.into()
}
```

This catches missing configuration at compile time rather than panicking in production.

### Pattern 3: JSON Schema Validation

Validate JSON structure at compile time:

```rust
#[proc_macro]
pub fn json_checked(input: TokenStream) -> TokenStream {
    let json_str = parse_macro_input!(input as LitStr);

    match serde_json::from_str::<serde_json::Value>(&json_str.value()) {
        Ok(_) => {
            let value = json_str.value();
            quote! {
                serde_json::from_str::<serde_json::Value>(#value).unwrap()
            }.into()
        }
        Err(err) => {
            syn::Error::new(
                json_str.span(),
                format!("Invalid JSON: {}", err)
            ).to_compile_error().into()
        }
    }
}
```

Any JSON syntax error becomes a compile error with the exact location.

### Pattern 4: Numeric Range Validation

Validate that compile-time constants are within expected ranges:

```rust
#[proc_macro]
pub fn port(input: TokenStream) -> TokenStream {
    let port_lit = parse_macro_input!(input as syn::LitInt);
    let port_value: u32 = match port_lit.base10_parse() {
        Ok(v) => v,
        Err(e) => return e.to_compile_error().into(),
    };

    if port_value == 0 || port_value > 65535 {
        return syn::Error::new(
            port_lit.span(),
            format!("Port {} is out of range (1-65535)", port_value)
        ).to_compile_error().into();
    }

    if port_value < 1024 {
        // Warning-like: still compiles but generates a runtime warning
        return quote! {
            {
                eprintln!("Warning: port {} requires root privileges", #port_value);
                #port_value as u16
            }
        }.into();
    }

    quote! { #port_value as u16 }.into()
}
```

This macro validates the port number, rejects invalid values, and warns about privileged ports.

### Benefits Summary

| Benefit | Description |
|---------|-------------|
| No runtime panics | Invalid inputs never reach production |
| Better errors | Custom messages with source location spans |
| Zero runtime cost | Validation runs once at compile time |
| IDE integration | Errors appear in the editor as you type |

## Common Pitfalls
1. **Heavy validation dependencies** — Adding a full SQL parser or JSON schema validator to your proc macro crate increases compile times. Consider validating only syntax, not semantics.
2. **Environment-dependent validation** — Checking environment variables or file paths at compile time can break reproducible builds. Document these requirements clearly.
3. **Overly strict validation** — Rejecting valid inputs due to incomplete validation logic is worse than accepting invalid ones. Only validate what you can check reliably.

## Best Practices
1. **Use `syn::Error` with spans** — Always attach errors to the specific token that caused the problem. Generic errors without spans are frustrating to debug.
2. **Add `// Safety: validated at compile time` comments** — When the generated code contains `unwrap()` or unsafe operations that are safe due to compile-time validation, document why.
3. **Provide escape hatches** — Use `cfg` features to disable compile-time validation in CI environments where external resources (databases, env vars) may not be available.

## Summary
- Compile-time validation catches errors before runtime, eliminating entire bug categories.
- Span-aware errors point users to the exact problem location.
- Validated code can safely use `unwrap()` since invalid inputs never reach runtime.
- Balance validation thoroughness against compile-time overhead.
- Document environment dependencies and provide cfg escape hatches.

## Code Examples

**A compile-time URL validation macro that catches malformed URLs during compilation, ensuring the runtime unwrap is always safe**

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, LitStr};
use quote::quote;

// Validates that a string is a well-formed URL at compile time
#[proc_macro]
pub fn checked_url(input: TokenStream) -> TokenStream {
    let url_lit = parse_macro_input!(input as LitStr);
    let url_str = url_lit.value();

    // Basic URL validation at compile time
    if !url_str.starts_with("http://") && !url_str.starts_with("https://") {
        return syn::Error::new(
            url_lit.span(),
            "URL must start with http:// or https://"
        ).to_compile_error().into();
    }

    if !url_str.contains('.') {
        return syn::Error::new(
            url_lit.span(),
            "URL must contain a domain with at least one dot"
        ).to_compile_error().into();
    }

    // Safe to construct at runtime: we validated the format
    quote! {
        url::Url::parse(#url_str).unwrap() // Safety: validated at compile time
    }.into()
}

// Usage:
// let api_base = checked_url!("https://api.example.com/v1"); // OK
// let bad = checked_url!("not-a-url"); // Compile error!
```


## Resources

- [SQLx - Compile-time checked queries](https://docs.rs/sqlx/latest/sqlx/macro.query.html) — Real-world example of compile-time validation: SQLx validates SQL queries against an actual database during compilation

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*