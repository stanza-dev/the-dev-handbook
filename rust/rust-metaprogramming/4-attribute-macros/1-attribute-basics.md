---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-attribute-basics"
---

# Attribute Macros

Attribute macros receive and can modify the item they're attached to.

```rust
#[proc_macro_attribute]
pub fn my_attr(
    args: TokenStream,   // Arguments: #[my_attr(these, args)]
    input: TokenStream,  // The item being annotated
) -> TokenStream {
    // Return modified or new item
}
```

## Example: Function Timing

```rust
#[proc_macro_attribute]
pub fn timed(_args: TokenStream, input: TokenStream) -> TokenStream {
    let input_fn = parse_macro_input!(input as syn::ItemFn);
    
    let fn_name = &input_fn.sig.ident;
    let fn_block = &input_fn.block;
    let fn_vis = &input_fn.vis;
    let fn_sig = &input_fn.sig;
    
    let output = quote! {
        #fn_vis #fn_sig {
            let __start = std::time::Instant::now();
            let __result = { #fn_block };
            println!("{} took {:?}", stringify!(#fn_name), __start.elapsed());
            __result
        }
    };
    
    output.into()
}
```

## Usage

```rust
#[timed]
fn expensive_operation() -> i32 {
    std::thread::sleep(std::time::Duration::from_millis(100));
    42
}

fn main() {
    let result = expensive_operation();
    // Prints: "expensive_operation took 100.xxxms"
}
```

## Key Differences from Derive

| Feature | Derive | Attribute |
|---------|--------|----------|
| Can modify input | No | Yes |
| Can remove input | No | Yes |
| Gets arguments | Limited | Full |
| Where attached | struct/enum | Any item |

## Code Examples

**Attribute with arguments**

```rust
// Attribute macro that adds logging
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, ItemFn, AttributeArgs, NestedMeta, Lit};

#[proc_macro_attribute]
pub fn log_call(args: TokenStream, input: TokenStream) -> TokenStream {
    let args = parse_macro_input!(args as AttributeArgs);
    let input_fn = parse_macro_input!(input as ItemFn);
    
    // Parse optional level argument: #[log_call(level = "debug")]
    let level = args.iter().find_map(|arg| {
        if let NestedMeta::Meta(syn::Meta::NameValue(nv)) = arg {
            if nv.path.is_ident("level") {
                if let Lit::Str(s) = &nv.lit {
                    return Some(s.value());
                }
            }
        }
        None
    }).unwrap_or_else(|| "info".to_string());
    
    let fn_name = &input_fn.sig.ident;
    let fn_block = &input_fn.block;
    let fn_vis = &input_fn.vis;
    let fn_sig = &input_fn.sig;
    
    let output = quote! {
        #fn_vis #fn_sig {
            log::log!(log::Level::from_str(#level).unwrap(), 
                      "Calling {}", stringify!(#fn_name));
            let result = { #fn_block };
            log::log!(log::Level::from_str(#level).unwrap(),
                      "{} returned", stringify!(#fn_name));
            result
        }
    };
    
    output.into()
}

// Usage:
#[log_call(level = "debug")]
fn my_function() -> i32 { 42 }
```


---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*