---
source_course: "rust"
source_lesson: "rust-variables-and-mutability"
---

# Variables, Mutability & Shadowing

## Introduction

Rust takes an opinionated stance on variables: they are immutable by default. This might feel restrictive if you come from languages like JavaScript or Python, but it is a deliberate design choice that makes code safer and easier to reason about. Understanding mutability, constants, and shadowing is essential before you write any real Rust code.

## Key Concepts

- **Immutable variable**: A binding declared with `let` that cannot be reassigned. This is the default in Rust.
- **Mutable variable**: A binding declared with `let mut` that can be reassigned to a new value of the same type.
- **Constant**: A value declared with `const` that is always immutable, requires a type annotation, and must be a compile-time expression.
- **Shadowing**: Declaring a new variable with the same name as a previous one using `let`, which creates a fresh binding that can even change the type.

## Real World Context

Immutability by default prevents an entire category of bugs where a variable is accidentally modified in a distant part of the code. In a large codebase, seeing `let mut` immediately signals that a value will change, making code reviews faster and intent clearer. Shadowing is used constantly in real Rust code for transforming values through a pipeline while keeping meaningful names.

## Deep Dive

In Rust, variables are immutable by default. The compiler rejects any attempt to reassign them:

```rust
let x = 5;
// x = 6; // Error! Cannot assign twice to immutable variable
```

To opt into mutability, add the `mut` keyword:

```rust
let mut y = 10;
y = 15; // OK
y += 1; // Also OK
```

Constants are different from immutable variables in several important ways:

```rust
const MAX_POINTS: u32 = 100_000; // MUST have type annotation
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3; // Computed at compile time
```

Constants require type annotations, can only be set to constant expressions, can never use `mut`, and can be declared in any scope including global scope.

Shadowing is one of Rust's most powerful patterns. You can declare a new variable with the same name using `let`, which creates an entirely new binding:

```rust
let spaces = "   ";        // &str
let spaces = spaces.len(); // usize - type changed!
```

This is different from mutation. With `mut`, you cannot change the type:

```rust
let mut spaces = "   ";
// spaces = spaces.len(); // Error! Can't change type with mut
```

Shadowing is especially useful when transforming a value through several steps or converting types while keeping a meaningful name:

```rust
let guess: u32 = "42".parse().expect("Not a number");
```

## Common Pitfalls

1. **Confusing shadowing with mutation** — Shadowing creates a completely new variable that happens to have the same name. It can change the type and does not require `mut`. Mutation modifies the existing variable in place and cannot change its type.
2. **Forgetting `mut` when you need it** — If you need to modify a variable, declare it with `let mut` from the start. The compiler error message will remind you, but anticipating it saves time.
3. **Using `mut` when shadowing would be cleaner** — If you only need to transform a value once, shadowing with `let` is more idiomatic than using `let mut` and reassigning.

## Best Practices

1. **Default to immutability** — Only add `mut` when you genuinely need to modify the variable. This makes your intent explicit and helps the compiler optimize.
2. **Use shadowing for type transformations** — When parsing a string into a number or processing data through stages, shadow the variable rather than inventing new names like `input_str` and `input_num`.

## Summary

- Variables in Rust are immutable by default; use `let mut` to opt into mutability.
- Constants (`const`) require type annotations and must be compile-time expressions.
- Shadowing with `let` creates a new variable, allowing type changes unlike `mut`.
- Immutability by default makes code safer and intent clearer.
- Prefer shadowing over mutation when transforming a value through stages.

## Code Examples

**Shadowing in nested scopes**

```rust
fn main() {
    let x = 5;
    let x = x + 1; // Shadow with new value
    {
        let x = x * 2; // Inner scope shadow
        println!("Inner x: {x}"); // 12
    }
    println!("Outer x: {x}"); // 6
}
```


## Resources

- [Variables and Mutability](https://doc.rust-lang.org/stable/book/ch03-01-variables-and-mutability.html) — Official Rust Book chapter on variables, mutability, and shadowing
- [Constants and Statics](https://doc.rust-lang.org/reference/items/constant-items.html) — Rust Reference on const and static items

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*