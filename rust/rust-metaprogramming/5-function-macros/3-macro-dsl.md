---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-function-macro-dsl"
---

# Building Domain-Specific Languages

## Introduction
Function-like proc macros are the foundation for embedding domain-specific languages (DSLs) within Rust. A DSL provides syntax and semantics tailored to a specific problem domain — like HTML templates, routing tables, or state machines — while still compiling to efficient Rust code. By implementing custom `Parse` types, you can accept syntax that is not valid Rust and transform it into type-safe, optimized output.

## Key Concepts
- **DSL (Domain-Specific Language)**: A focused language for a specific domain, embedded in Rust through a macro.
- **Custom parser**: An implementation of `syn::parse::Parse` that defines the grammar of your DSL.
- **`ParseStream` methods**: `parse()`, `peek()`, `lookahead1()`, `fork()` provide the tools for building recursive descent parsers.
- **AST transformation**: Parsing the DSL into an intermediate representation, then generating Rust code from that representation.

## Real World Context
Yew's `html!` macro provides JSX-like syntax for building web UIs. Diesel's `table!` macro defines database schema as a DSL. Pest's `grammar!` macro defines PEG parsers. These DSLs dramatically reduce boilerplate while maintaining Rust's safety guarantees. Building a DSL is one of the most advanced and rewarding proc macro techniques.

## Deep Dive

### Designing the Grammar

Before writing code, define what syntax your DSL accepts. For a routing DSL:

```rust
// Desired syntax:
routes! {
    GET "/users" => list_users,
    POST "/users" => create_user,
    GET "/users/:id" => get_user,
    DELETE "/users/:id" => delete_user,
}
```

This is not valid Rust, but we can parse it with a custom parser.

### Step 1: Define AST Types

```rust
struct Route {
    method: Ident,
    path: LitStr,
    handler: Ident,
}

struct RoutesInput {
    routes: Vec<Route>,
}
```

These types represent the parsed structure of the DSL.

### Step 2: Implement Parsers

```rust
impl Parse for Route {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let method: Ident = input.parse()?;

        // Validate HTTP method
        let method_str = method.to_string();
        if !["GET", "POST", "PUT", "DELETE", "PATCH"]
            .contains(&method_str.as_str()) {
            return Err(syn::Error::new(
                method.span(),
                format!("'{}' is not a valid HTTP method", method_str)
            ));
        }

        let path: LitStr = input.parse()?;
        input.parse::<Token![=>]>()?;
        let handler: Ident = input.parse()?;

        Ok(Route { method, path, handler })
    }
}

impl Parse for RoutesInput {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let routes = Punctuated::<Route, Token![,]>::parse_terminated(input)?;
        Ok(RoutesInput {
            routes: routes.into_iter().collect(),
        })
    }
}
```

The `Punctuated::parse_terminated` helper handles comma-separated items with an optional trailing comma.

### Step 3: Generate Code

Transform the parsed routes into Rust code:

```rust
#[proc_macro]
pub fn routes(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as RoutesInput);

    let route_registrations = input.routes.iter().map(|route| {
        let method = route.method.to_string().to_lowercase();
        let method_ident = Ident::new(&method, route.method.span());
        let path = &route.path;
        let handler = &route.handler;

        quote! {
            router.#method_ident(#path, #handler);
        }
    });

    quote! {
        {
            let mut router = Router::new();
            #(#route_registrations)*
            router
        }
    }.into()
}
```

The DSL compiles into standard router method calls.

### Advanced: Lookahead and Error Recovery

For complex DSLs, use `lookahead1()` to provide better error messages:

```rust
impl Parse for ConfigValue {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let lookahead = input.lookahead1();

        if lookahead.peek(LitStr) {
            let value: LitStr = input.parse()?;
            Ok(ConfigValue::String(value.value()))
        } else if lookahead.peek(LitInt) {
            let value: LitInt = input.parse()?;
            Ok(ConfigValue::Int(value.base10_parse()?))
        } else if lookahead.peek(LitBool) {
            let value: LitBool = input.parse()?;
            Ok(ConfigValue::Bool(value.value()))
        } else {
            // lookahead1 generates: "expected string literal, integer, or boolean"
            Err(lookahead.error())
        }
    }
}
```

The `lookahead1()` method peeks at the next token without consuming it, and its `error()` method lists all the types you checked, producing a message like "expected string literal, integer literal, or boolean literal".

## Common Pitfalls
1. **Ambiguous grammar** — If two parse branches can match the same prefix, the parser may take the wrong path. Use `peek()` to look ahead before committing.
2. **Poor error messages** — A bare `input.parse::<SomeType>()?` produces a generic error. Use `lookahead1()` or custom errors for better diagnostics.
3. **Untestable parsers** — If your parser is only callable from the proc macro entry point, you cannot write unit tests. Accept `proc_macro2::TokenStream` in helper functions.

## Best Practices
1. **Parse into a clean AST, then generate** — Separate parsing from code generation. This makes both easier to test and maintain.
2. **Use `Punctuated` for lists** — The `syn::punctuated::Punctuated` type handles comma-separated lists with optional trailing separators.
3. **Validate semantics after parsing** — Parse the syntax first, then validate semantic rules (like "no duplicate routes") in a separate pass with better error context.

## Summary
- Function-like proc macros enable embedded DSLs with custom syntax.
- Design the grammar first, then implement `Parse` traits for each construct.
- Use `Punctuated` for comma-separated lists and `lookahead1()` for clear errors.
- Separate parsing, validation, and code generation into distinct phases.
- Test parsers with `proc_macro2::TokenStream` outside the macro runtime.

## Code Examples

**A state machine DSL that parses custom transition syntax and generates State/Event enums with a transition function**

```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, parse::{Parse, ParseStream}, Token,
         Ident, LitStr, LitInt, punctuated::Punctuated};
use quote::{quote, format_ident};

// DSL for defining a state machine:
// state_machine! {
//     Idle => Start -> Running,
//     Running => Stop -> Idle,
//     Running => Pause -> Paused,
//     Paused => Resume -> Running,
// }

struct Transition {
    from: Ident,
    event: Ident,
    to: Ident,
}

impl Parse for Transition {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let from: Ident = input.parse()?;
        input.parse::<Token![=>]>()?;
        let event: Ident = input.parse()?;
        input.parse::<Token![->]>()?;
        let to: Ident = input.parse()?;
        Ok(Transition { from, event, to })
    }
}

struct StateMachine {
    transitions: Vec<Transition>,
}

impl Parse for StateMachine {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let transitions =
            Punctuated::<Transition, Token![,]>::parse_terminated(input)?;
        Ok(StateMachine {
            transitions: transitions.into_iter().collect(),
        })
    }
}

#[proc_macro]
pub fn state_machine(input: TokenStream) -> TokenStream {
    let sm = parse_macro_input!(input as StateMachine);
    let states: Vec<_> = sm.transitions.iter()
        .flat_map(|t| vec![&t.from, &t.to])
        .collect::<std::collections::HashSet<_>>()
        .into_iter().collect();

    let match_arms = sm.transitions.iter().map(|t| {
        let from = &t.from;
        let event = &t.event;
        let to = &t.to;
        quote! {
            (State::#from, Event::#event) => State::#to,
        }
    });

    let events: Vec<_> = sm.transitions.iter()
        .map(|t| &t.event)
        .collect::<std::collections::HashSet<_>>()
        .into_iter().collect();

    quote! {
        #[derive(Debug, Clone, PartialEq)]
        enum State { #(#states,)* }

        #[derive(Debug, Clone)]
        enum Event { #(#events,)* }

        fn transition(state: State, event: Event) -> State {
            match (state, event) {
                #(#match_arms)*
                (state, _) => state,  // No-op for undefined transitions
            }
        }
    }.into()
}
```


## Resources

- [Function-like Procedural Macros - The Rust Reference](https://doc.rust-lang.org/reference/procedural-macros.html#function-like-procedural-macros) — Official reference for function-like procedural macro definitions and invocation
- [proc-macro-workshop](https://github.com/dtolnay/proc-macro-workshop) — Hands-on exercises for building real-world proc macros including custom syntax parsing and DSL design

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*