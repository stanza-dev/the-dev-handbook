---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-fragment-specifiers"
---

# Fragment Specifiers Deep Dive

## Introduction
Fragment specifiers are the type system for macro patterns. Each specifier tells the compiler exactly what kind of Rust syntax a `$name:spec` placeholder can capture. Choosing the right specifier is essential for writing macros that are both flexible and correct. Rust 1.94 (2024 edition) introduces notable changes to the `expr` specifier and adds `expr_2021` for backwards compatibility.

## Key Concepts
- **Fragment specifier**: The keyword after the colon in `$name:specifier` that constrains what tokens can be captured.
- **`expr` (2024 edition)**: Now matches underscore expressions (`_`) and `const { }` blocks in addition to all previous expression forms.
- **`expr_2021`**: A backwards-compatible specifier introduced in the 2024 edition that matches the same set of expressions as `expr` did in the 2021 edition — it excludes `_` (UnderscoreExpression) and `const {}` (ConstBlockExpression).
- **`pat_param`**: Matches the pre-2021 pattern syntax (no top-level alternations). Useful when you need the older, more restrictive pattern matching.
- **`tt` (token tree)**: The most permissive specifier — matches any single token or balanced group. Use it as a last resort.

## Real World Context
Understanding fragment specifiers prevents cryptic compiler errors when users invoke your macro with unexpected syntax. The 2024 edition changes to `expr` matter because macros that previously rejected `const { }` blocks as expressions now accept them. If your macro needs the older behavior (for example, to avoid ambiguity in a custom parser), use `expr_2021` instead. Libraries maintaining backwards compatibility across editions will find `expr_2021` and `pat_param` essential.

## Deep Dive

Here is the complete table of fragment specifiers available in Rust 1.94:

| Specifier | Matches | Example |
|-----------|---------|----------|
| `expr` | Expressions (2024: includes `_` and `const {}`) | `x + 1`, `const { 42 }` |
| `expr_2021` | Expressions (2021 rules, no `_` or `const {}`) | `x + 1`, `foo()` |
| `ident` | Identifiers | `my_var`, `String` |
| `ty` | Types | `i32`, `Vec<String>` |
| `pat` | Patterns (2021+: allows top-level `\|`) | `Some(x)`, `1 \| 2` |
| `pat_param` | Patterns (pre-2021, no top-level `\|`) | `Some(x)`, `ref y` |
| `path` | Paths | `std::io::Result` |
| `stmt` | Statements | `let x = 1` |
| `block` | Blocks | `{ x + 1 }` |
| `item` | Items | `fn foo() {}`, `struct Bar;` |
| `literal` | Literals | `42`, `"hello"`, `true` |
| `tt` | Token tree (any single token or balanced group) | anything |
| `meta` | Attribute content | `derive(Debug)` |
| `lifetime` | Lifetimes | `'a`, `'static` |
| `vis` | Visibility qualifiers | `pub`, `pub(crate)`, (empty) |

The distinction between `expr` and `expr_2021` matters when your macro follows the captured expression with another token. In the 2024 edition, `expr` accepts `const { }` blocks, which changes what tokens can follow the capture:

```rust
// In 2024 edition, this macro accepts const blocks as expressions
macro_rules! eval {
    ($e:expr) => {
        println!("Result: {:?}", $e);
    };
}

// All of these work in 2024 edition:
eval!(42);
eval!(2 + 2);
eval!(const { 100 + 200 }); // NEW in 2024 edition
```

The `const { }` expression syntax lets you evaluate expressions at compile time inline. Since Rust 1.87, standard macros like `assert_eq!` and `vec!` support `const { }` arguments natively.

If you need backwards-compatible behavior that rejects these new expression forms, use `expr_2021`:

```rust
macro_rules! legacy_eval {
    ($e:expr_2021) => {
        println!("Result: {:?}", $e);
    };
}

legacy_eval!(42);            // Works
// legacy_eval!(const { 42 }); // Rejected by expr_2021
```

Similarly, `pat_param` restricts pattern matching to the pre-2021 form (no top-level `|` alternation), which is useful in function parameter positions where alternation would be ambiguous:

```rust
macro_rules! match_value {
    ($val:expr, $p:pat_param => $body:expr) => {
        match $val {
            $p => $body,
            _ => panic!("no match"),
        }
    };
}

match_value!(Some(42), Some(n) => n); // Returns 42
```

The `pat_param` specifier captures `Some(n)` without allowing `|` at the top level, which prevents parsing ambiguity with the `=>` that follows.

## Common Pitfalls
1. **Using `expr` when you need `expr_2021`** — If your macro follows a captured expression with a token that conflicts with `const { }` block syntax, the 2024 edition `expr` may cause unexpected parsing. Use `expr_2021` for the old behavior.
2. **Using `tt` as a lazy default** — While `tt` matches everything, it provides no guarantees about what was captured. Prefer a specific specifier so the compiler validates the input for you.
3. **Forgetting `vis` can be empty** — The `vis` specifier matches `pub`, `pub(crate)`, or nothing at all. This is useful but surprising if you expect it to always consume tokens.

## Best Practices
1. **Use the most specific specifier possible** — `ident` over `tt`, `expr` over `tt`, etc. The compiler gives better error messages when a specifier rejects bad input.
2. **Test edition-specific behavior** — If your macro is a library used across editions, test with both `expr` and `expr_2021` to ensure compatibility.
3. **Document accepted syntax** — Add doc comments to your macros showing what each arm expects, especially when using less common specifiers like `meta` or `vis`.

## Summary
- Rust 1.94 (2024 edition) expands `expr` to include `_` and `const { }` blocks.
- `expr_2021` preserves the old behavior for backwards compatibility.
- `pat_param` captures patterns without top-level `|` alternation (pre-2021 rules).
- Always choose the most specific specifier that fits your use case.
- `tt` is the escape hatch but should be a last resort.

## Code Examples

**Each specifier captures a different kind of Rust syntax, and the compiler validates the input accordingly**

```rust
// Demonstrating specifier differences
macro_rules! show_type {
    (ident: $i:ident) => {
        println!("Identifier: {}", stringify!($i));
    };
    (expr: $e:expr) => {
        println!("Expression value: {:?}", $e);
    };
    (ty: $t:ty) => {
        println!("Type: {}", stringify!($t));
    };
    (path: $p:path) => {
        println!("Path: {}", stringify!($p));
    };
}

show_type!(ident: my_variable);        // "Identifier: my_variable"
show_type!(expr: 2 + 3);               // "Expression value: 5"
show_type!(ty: Vec<String>);            // "Type: Vec<String>"
show_type!(path: std::collections::HashMap); // "Path: std::collections::HashMap"
```


## Resources

- [Fragment Specifiers - The Rust Reference](https://doc.rust-lang.org/reference/macros-by-example.html#metavariables) — Official reference listing all fragment specifiers and their matching rules

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*