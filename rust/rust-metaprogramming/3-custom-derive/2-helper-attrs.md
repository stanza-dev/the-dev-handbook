---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-derive-helper-attrs"
---

# Helper Attributes in Derive Macros

## Introduction
Helper attributes let derive macro users annotate individual fields to customize the generated code. When you write `#[derive(Serialize)]` and then mark a field with `#[serde(skip)]`, that `skip` attribute is a helper attribute. This mechanism bridges the gap between the all-or-nothing nature of derive macros and the fine-grained control that real-world code generation requires.

## Key Concepts
- **Helper attribute**: An attribute recognized by a derive macro, declared in the `attributes(...)` parameter of `#[proc_macro_derive]`.
- **`#[proc_macro_derive(Name, attributes(helper))]`**: Registers `helper` as a recognized attribute, preventing the compiler from rejecting it as unknown.
- **`Attribute::parse_nested_meta()`**: The syn 2.0 API for parsing structured attribute arguments like `#[helper(key = "value")]`.

## Real World Context
Serde's derive macros use helper attributes extensively: `#[serde(rename = "name")]`, `#[serde(skip)]`, `#[serde(default)]`. Clap uses `#[arg(short, long)]`. These attributes give users control over per-field behavior without requiring a separate macro for each customization. Any derive macro that needs field-level configuration uses this pattern.

## Deep Dive

### Declaring Helper Attributes

Register helper attributes in the derive macro declaration:

```rust
#[proc_macro_derive(MySerialize, attributes(my_serde))]
pub fn my_serialize_derive(input: TokenStream) -> TokenStream {
    // Fields can now use #[my_serde(...)] without compiler warnings
    todo!()
}
```

Without the `attributes(my_serde)` declaration, the compiler would reject `#[my_serde(skip)]` as an unrecognized attribute.

### Parsing Helper Attributes with syn 2.0

In syn 2.0, you parse attribute arguments using `Attribute::parse_nested_meta()`. This replaces the removed `AttributeArgs` and `NestedMeta` types from syn 1.x:

```rust
use syn::{Attribute, Ident};

struct FieldConfig {
    skip: bool,
    rename: Option<String>,
}

fn parse_field_attrs(attrs: &[Attribute]) -> syn::Result<FieldConfig> {
    let mut config = FieldConfig {
        skip: false,
        rename: None,
    };

    for attr in attrs {
        if !attr.path().is_ident("my_serde") {
            continue; // Not our attribute, skip
        }

        attr.parse_nested_meta(|meta| {
            if meta.path.is_ident("skip") {
                config.skip = true;
                Ok(())
            } else if meta.path.is_ident("rename") {
                let value = meta.value()?; // parse the `=`
                let lit: syn::LitStr = value.parse()?;
                config.rename = Some(lit.value());
                Ok(())
            } else {
                Err(meta.error("unrecognized my_serde attribute"))
            }
        })?;
    }

    Ok(config)
}
```

The `parse_nested_meta` method calls your closure once for each nested item inside the attribute parentheses. The closure receives a `ParseNestedMeta` value that provides the path and methods to parse values.

### Using Field Configs in Code Generation

With parsed configs, you can conditionally generate code per field:

```rust
let field_serializers = fields.named.iter().map(|f| {
    let config = parse_field_attrs(&f.attrs).expect("valid attrs");
    let field_name = &f.ident;

    if config.skip {
        return quote! {}; // Skip this field entirely
    }

    let serialized_name = config.rename
        .unwrap_or_else(|| field_name.as_ref().unwrap().to_string());

    quote! {
        map.insert(
            #serialized_name.to_string(),
            format!("{:?}", self.#field_name)
        );
    }
});
```

Fields marked `#[my_serde(skip)]` produce no code, and renamed fields use the custom name in the output.

### Complete Example

Here is how a user would use the derive macro with helper attributes:

```rust
#[derive(MySerialize)]
struct UserProfile {
    #[my_serde(rename = "user_name")]
    username: String,

    email: String,

    #[my_serde(skip)]
    password_hash: String,
}

// Generated serialize() method would:
// - serialize username as "user_name"
// - serialize email as "email"
// - skip password_hash entirely
```

## Common Pitfalls
1. **Forgetting to declare helper attributes** — Without `attributes(my_serde)` in the derive declaration, users get "unknown attribute" errors.
2. **Using syn 1.x API** — `AttributeArgs` and `NestedMeta` were removed in syn 2.0. Use `Attribute::parse_nested_meta()` instead.
3. **Not handling unknown attribute keys** — If a user writes `#[my_serde(unknown_key)]`, your parser should return a clear error rather than silently ignoring it.

## Best Practices
1. **Return `syn::Result` from attribute parsers** — Propagate errors with `?` so the user sees a clear message pointing at the problematic attribute.
2. **Validate combinations** — Check for conflicting attributes (e.g., `skip` and `rename` on the same field) and produce a descriptive error.
3. **Document your helper attributes** — Add doc comments or a README listing all supported attribute keys and their effects.

## Summary
- Helper attributes let derive macro users customize per-field behavior.
- Declare them with `#[proc_macro_derive(Name, attributes(helper))]`.
- Parse arguments with `Attribute::parse_nested_meta()` in syn 2.0.
- Always handle unknown attribute keys with clear error messages.
- `AttributeArgs` and `NestedMeta` from syn 1.x are removed; do not use them.

## Code Examples

**A derive macro with a helper attribute #[my_serde(skip)] using syn 2.0's parse_nested_meta API to conditionally skip fields during serialization**

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput, Data, Fields, Attribute, Error};
use quote::quote;

#[proc_macro_derive(MySerialize, attributes(my_serde))]
pub fn serialize_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    match generate_serialize(&input) {
        Ok(tokens) => tokens.into(),
        Err(err) => err.to_compile_error().into(),
    }
}

fn should_skip(attrs: &[Attribute]) -> syn::Result<bool> {
    let mut skip = false;
    for attr in attrs {
        if attr.path().is_ident("my_serde") {
            attr.parse_nested_meta(|meta| {
                if meta.path.is_ident("skip") {
                    skip = true;
                    Ok(())
                } else {
                    Err(meta.error("expected `skip`"))
                }
            })?;
        }
    }
    Ok(skip)
}

fn generate_serialize(input: &DeriveInput) -> Result<proc_macro2::TokenStream, Error> {
    let name = &input.ident;
    let fields = match &input.data {
        Data::Struct(data) => match &data.fields {
            Fields::Named(f) => &f.named,
            _ => return Err(Error::new_spanned(name, "expected named fields")),
        },
        _ => return Err(Error::new_spanned(name, "expected a struct")),
    };

    let serializers = fields.iter().map(|f| {
        let field_name = &f.ident;
        let key = field_name.as_ref().unwrap().to_string();
        if should_skip(&f.attrs).unwrap_or(false) {
            quote! {} // skip this field
        } else {
            quote! { map.insert(#key.to_string(), format!("{:?}", self.#field_name)); }
        }
    });

    Ok(quote! {
        impl #name {
            pub fn serialize(&self) -> std::collections::HashMap<String, String> {
                let mut map = std::collections::HashMap::new();
                #(#serializers)*
                map
            }
        }
    })
}
```


## Resources

- [syn Attribute parsing](https://docs.rs/syn/2/syn/struct.Attribute.html#method.parse_nested_meta) — syn 2.0 API reference for parse_nested_meta, the replacement for the removed AttributeArgs and NestedMeta types

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*