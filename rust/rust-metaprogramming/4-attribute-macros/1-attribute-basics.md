---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-attribute-basics"
---

# Attribute Macro Fundamentals

## Introduction
Attribute macros are the most powerful type of procedural macro. Unlike derive macros that can only add code, attribute macros receive the full annotated item and can modify, replace, or augment it. They are defined with `#[proc_macro_attribute]` and receive two token streams: the attribute arguments and the item being annotated. This makes them ideal for cross-cutting concerns like logging, timing, access control, and test frameworks.

## Key Concepts
- **`#[proc_macro_attribute]`**: The attribute marking a function as an attribute macro. The function receives two `TokenStream` parameters.
- **`args` parameter**: The tokens inside the attribute parentheses: `#[my_attr(these_tokens)]`.
- **`input` parameter**: The complete item (function, struct, impl block) the attribute is attached to.
- **Item replacement**: The return value *replaces* the original item entirely. To keep the original item, you must include it in your output.

## Real World Context
Attribute macros power many of Rust's most important libraries. Tokio's `#[tokio::main]` transforms a `main` function into an async runtime entry point. Actix-web's `#[get("/path")]` turns functions into route handlers. Test frameworks use `#[test]` attributes. Whenever you need to wrap, modify, or transform an item based on annotations, an attribute macro is the right tool.

## Deep Dive

### The Two-Parameter Signature

Every attribute macro has this signature:

```rust
#[proc_macro_attribute]
pub fn my_attribute(
    args: TokenStream,   // Arguments: #[my_attribute(these, args)]
    input: TokenStream,  // The annotated item
) -> TokenStream {
    // Return the modified or new item
}
```

The `args` stream contains only what is inside the parentheses. The `input` stream contains the entire item, including any other attributes on it.

### Example: Function Timing

A practical attribute macro that wraps a function body to measure execution time:

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, ItemFn};
use quote::quote;

#[proc_macro_attribute]
pub fn timed(_args: TokenStream, input: TokenStream) -> TokenStream {
    let input_fn = parse_macro_input!(input as ItemFn);

    let fn_name = &input_fn.sig.ident;
    let fn_block = &input_fn.block;
    let fn_vis = &input_fn.vis;
    let fn_sig = &input_fn.sig;
    let fn_attrs = &input_fn.attrs;

    let output = quote! {
        #(#fn_attrs)*
        #fn_vis #fn_sig {
            let __start = std::time::Instant::now();
            let __result = (|| #fn_block)();
            eprintln!("{} took {:?}", stringify!(#fn_name), __start.elapsed());
            __result
        }
    };

    output.into()
}
```

This macro preserves the function's visibility, signature, and other attributes while wrapping the body in timing instrumentation. The `(|| #fn_block)()` pattern ensures the original block's return value is captured correctly.

### Usage

```rust
#[timed]
fn expensive_computation() -> i32 {
    std::thread::sleep(std::time::Duration::from_millis(100));
    42
}

fn main() {
    let result = expensive_computation();
    // Prints: "expensive_computation took 100.xxxms"
    println!("Result: {}", result);  // Output: Result: 42
}
```

The user's function looks normal but gains timing behavior.

### Key Differences from Derive

| Feature | Derive | Attribute |
|---------|--------|----------|
| Can modify input | No (additive only) | Yes (full replacement) |
| Can remove input | No | Yes |
| Gets arguments | Only helper attrs | Full argument stream |
| Where attached | struct/enum only | Any item (fn, struct, impl, mod) |
| Number of inputs | 1 TokenStream | 2 TokenStreams (args + item) |

The critical difference is that attribute macros *replace* the annotated item. If you want to keep the original item, you must emit it as part of your output.

## Common Pitfalls
1. **Forgetting to emit the original item** — The return value replaces the annotated item. If you only return new code without the original function, the function disappears.
2. **Not preserving other attributes** — The input item may have `#[inline]`, `#[cfg(...)]`, or other attributes. Always include `#(#fn_attrs)*` in the output to preserve them.
3. **Breaking return types** — When wrapping a function body, make sure your wrapper correctly forwards the return value. Use a closure or block that returns the original expression.

## Best Practices
1. **Parse `args` even if unused** — Validate that the user did not pass unexpected arguments by parsing the args stream and erroring on unexpected input.
2. **Preserve spans** — Use `quote_spanned!` to keep error messages pointing to the right location in the user's source code.
3. **Test with `#[cfg]` attributes** — Ensure your macro works correctly when the annotated item also has conditional compilation attributes.

## Summary
- Attribute macros receive two streams: arguments and the annotated item.
- They replace the original item entirely; include it in your output to preserve it.
- Use them for cross-cutting concerns like timing, logging, and access control.
- Always preserve other attributes and the function's return type.
- They work on any item, not just structs and enums.

## Code Examples

**An attribute macro that adds automatic retry logic to a function, demonstrating how to wrap a function body while preserving its signature and attributes**

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, ItemFn};
use quote::quote;

// Attribute macro that adds retry logic to a function
#[proc_macro_attribute]
pub fn retry(_args: TokenStream, input: TokenStream) -> TokenStream {
    let input_fn = parse_macro_input!(input as ItemFn);

    let fn_vis = &input_fn.vis;
    let fn_sig = &input_fn.sig;
    let fn_block = &input_fn.block;
    let fn_attrs = &input_fn.attrs;
    let fn_name = &input_fn.sig.ident;

    let output = quote! {
        #(#fn_attrs)*
        #fn_vis #fn_sig {
            let max_retries = 3;
            let mut attempt = 0;
            loop {
                attempt += 1;
                match std::panic::catch_unwind(std::panic::AssertUnwindSafe(|| #fn_block)) {
                    Ok(result) => break result,
                    Err(_) if attempt < max_retries => {
                        eprintln!("{}: attempt {} failed, retrying...",
                            stringify!(#fn_name), attempt);
                        continue;
                    }
                    Err(err) => std::panic::resume_unwind(err),
                }
            }
        }
    };

    output.into()
}

// Usage:
// #[retry]
// fn flaky_network_call() -> String {
//     // ... may panic on transient errors
// }
```


## Resources

- [Attribute Macros - The Rust Reference](https://doc.rust-lang.org/reference/procedural-macros.html#attribute-macros) — Official specification for attribute macro signatures, behavior, and resolution

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*