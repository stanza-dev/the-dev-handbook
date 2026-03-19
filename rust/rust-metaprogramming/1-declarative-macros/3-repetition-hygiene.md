---
source_course: "rust-metaprogramming"
source_lesson: "rust-meta-repetition-hygiene"
---

# Repetition & Hygiene

## Introduction
Repetition operators let macros accept variable numbers of arguments, and macro hygiene prevents generated code from accidentally clashing with the caller's variable names. Together, these features make declarative macros both powerful and safe. Understanding them is essential for writing macros like `vec!`, `println!`, or any macro that processes lists of items.

## Key Concepts
- **Repetition syntax**: `$( PATTERN )SEPARATOR QUANTIFIER` where the quantifier is `*` (zero or more), `+` (one or more), or `?` (zero or one).
- **Separator**: An optional token (commonly `,`) placed between repetitions.
- **Macro hygiene**: The compiler's mechanism for keeping variables defined inside a macro expansion separate from variables in the caller's scope.
- **Token tree munching**: A recursive pattern where the macro processes one token at a time, peeling off the head and recursing on the tail.

## Real World Context
Every macro that accepts a list of arguments uses repetition. The standard `vec![1, 2, 3]` uses `$($elem:expr),*` to capture a comma-separated list. The `println!` macro uses repetition for its format arguments. Understanding hygiene is crucial when your macro creates temporary variables — without it, a macro that generates `let temp = ...` could shadow the caller's own `temp` variable.

## Deep Dive

### Repetition Syntax

The repetition operator wraps a pattern in `$( )` followed by an optional separator and a quantifier:

```rust
macro_rules! make_struct {
    ($name:ident { $($field:ident: $ty:ty),* }) => {
        struct $name {
            $( $field: $ty, )*
        }
    };
}

make_struct!(Person { name: String, age: u32 });
// Expands to:
// struct Person {
//     name: String,
//     age: u32,
// }
```

The `$($field:ident: $ty:ty),*` captures zero or more comma-separated field declarations. In the expansion, `$( $field: $ty, )*` repeats once for each captured pair.

### Nested Repetition

Repetitions can nest to handle more complex structures:

```rust
macro_rules! nested {
    ($($outer:ident: [$($inner:expr),*]);*) => {
        $(
            let $outer = vec![$($inner),*];
        )*
    };
}

nested!(scores: [90, 85, 92]; grades: [4, 3]);
// Expands to:
// let scores = vec![90, 85, 92];
// let grades = vec![4, 3];
```

The outer repetition uses `;` as a separator, and each outer element contains an inner repetition with `,` as separator.

### Macro Hygiene

Rust's declarative macros are partially hygienic. Variables created inside a macro live in a different "syntax context" from the caller's scope:

```rust
macro_rules! make_x {
    () => {
        let x = 42;  // This x is hygienic
    };
}

fn main() {
    let x = 1;
    make_x!();
    println!("{x}");  // Still prints 1 - the macro's x is separate
}
```

The macro's `x` does not shadow the caller's `x` because they exist in different hygiene contexts. This prevents accidental name collisions.

However, when the caller passes an identifier or block into the macro, that code retains its original context and can interact with macro-defined variables:

```rust
macro_rules! with_value {
    ($body:block) => {
        {
            let value = 42;
            $body  // $body can reference 'value' because the macro defined it
        }
    };
}

let result = with_value!({ value + 8 }); // result = 50
```

The caller's block `{ value + 8 }` can see `value` because the caller explicitly chose to reference it.

### The Token Tree Muncher Pattern

This recursive technique processes tokens one at a time:

```rust
macro_rules! count_tokens {
    () => { 0usize };
    ($head:tt $($tail:tt)*) => {
        1usize + count_tokens!($($tail)*)
    };
}

assert_eq!(count_tokens!(a b c d), 4);
assert_eq!(count_tokens!(), 0);
```

Each recursive call peels off one token tree (`$head`) and recurses on the rest (`$tail`). The base case matches empty input and returns 0.

## Common Pitfalls
1. **Mismatched repetition counts** — In the expansion, every `$var` used together in a `$( )*` block must have been captured with the same repetition count. Mixing captures from different repetitions causes a compile error.
2. **Relying on hygiene for public APIs** — Hygiene only applies to `let` bindings and labels in `macro_rules!` macros, not to items like `fn`, `struct`, or `mod`. Generated item names are visible to the caller.
3. **Infinite recursion in tt munchers** — Always ensure the recursive pattern consumes at least one token per step, and always have a base case that matches empty input.

## Best Practices
1. **Use trailing comma support** — Add `$(,)?` at the end of comma-separated repetitions to accept an optional trailing comma: `$($e:expr),+ $(,)?`.
2. **Prefix internal names** — For generated items that should not clash with user code, use a naming convention like double underscores: `__internal_counter`.
3. **Limit recursion depth** — Rust has a default macro recursion limit of 128. For deeply recursive macros, add `#![recursion_limit = "256"]` at the crate root, but consider whether a procedural macro would be a better fit.

## Summary
- Repetition uses `$( )SEP QUANT` with `*`, `+`, or `?` quantifiers.
- Nested repetitions handle multi-level data structures.
- Macro hygiene keeps macro-internal variables separate from the caller's scope.
- Token tree munching enables recursive processing of arbitrary input.
- Always include a base case and ensure progress in recursive macros.

## Code Examples

**A hashmap! macro demonstrating repetition with custom separator (=>) and trailing comma support**

```rust
// A HashMap literal macro using repetition
macro_rules! hashmap {
    () => {
        ::std::collections::HashMap::new()
    };
    ($($key:expr => $value:expr),+ $(,)?) => {
        {
            let mut map = ::std::collections::HashMap::new();
            $(
                map.insert($key, $value);
            )+
            map
        }
    };
}

let empty: std::collections::HashMap<i32, i32> = hashmap!{};
let scores = hashmap!{
    "Alice" => 100,
    "Bob" => 95,
    "Charlie" => 87,
};

assert!(empty.is_empty());
assert_eq!(scores.get("Alice"), Some(&100));
assert_eq!(scores.len(), 3);
```


## Resources

- [Macros - The Rust Programming Language](https://doc.rust-lang.org/book/ch20-06-macros.html) — The Rust Book chapter on macros covering declarative and procedural approaches

---

> 📘 *This lesson is part of the [Rust Metaprogramming](https://stanza.dev/courses/rust-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*