---
source_course: "rust-traits"
source_lesson: "rust-traits-static-dispatch"
---

# Static Dispatch & Monomorphization

## Introduction
When you write generic code in Rust, the compiler generates specialized versions for each concrete type used. This process, called monomorphization, gives you the performance of hand-written code with the flexibility of generics.

## Key Concepts
- **Static Dispatch**: Method calls resolved at compile time based on the concrete type.
- **Monomorphization**: The compiler generates specialized code for each generic type instantiation.
- **Zero-Cost Abstraction**: Generics incur no runtime overhead because dispatch happens at compile time.

## Real World Context
Every time you use `Vec<T>`, `Option<T>`, or `Result<T, E>`, the compiler monomorphizes them. High-performance libraries like `serde` rely heavily on static dispatch to achieve near-zero overhead serialization.

## Deep Dive
When you define a generic function, Rust creates a separate copy for each concrete type it is called with:

```rust
fn print_it<T: Display>(item: T) {
    println!("{item}");
}

print_it(42);        // Generates print_it::<i32>
print_it("hello");   // Generates print_it::<&str>
print_it(3.14_f64);  // Generates print_it::<f64>
```

Each generated function is fully optimized for its specific type. The compiler can inline the concrete implementation and apply type-specific optimizations.

The key benefit is performance. There is no vtable lookup and no indirection. The call site knows exactly which function to invoke:

```rust
fn process<F: Fn(i32) -> i32>(f: F, x: i32) -> i32 {
    f(x)  // Can be inlined!
}

let double = |x| x * 2;
let result = process(double, 21); // Compiler inlines the closure
```

However, monomorphization has trade-offs. Each instantiation produces new machine code, which increases binary size and compile times. You also cannot create heterogeneous collections with static dispatch:

```rust
// This won't work: all elements must be the same type
// let items: Vec<???> = vec![42, "hello"]; // Error!
```

## Common Pitfalls
1. **Code bloat from excessive generics** — Making everything generic produces more monomorphized code. Only use generics where the flexibility is actually needed.
2. **Longer compile times** — Deeply nested generics (e.g., iterator chains) increase compile times significantly.

## Best Practices
1. **Use `impl Trait` in argument position for simple cases** — It reads more cleanly than explicit generic bounds: `fn foo(x: impl Display)`.
2. **Consider dynamic dispatch for large type sets** — If a function is called with many types, `dyn Trait` can reduce binary size at the cost of minor runtime overhead.

## Summary
- Static dispatch resolves method calls at compile time via monomorphization.
- The compiler generates specialized code for each concrete type, enabling inlining and optimization.
- Trade-offs include larger binaries and longer compile times.
- Static dispatch cannot support heterogeneous collections.

## Code Examples

**Static dispatch allows the compiler to specialize and inline iterator and closure calls, producing code as fast as hand-written loops**

```rust
// Trait bounds are zero-cost due to monomorphization
fn sum<I: Iterator<Item = i32>>(iter: I) -> i32 {
    iter.fold(0, |acc, x| acc + x)
}

let total = sum(vec![1, 2, 3].into_iter());
// Compiler specializes sum for Vec's IntoIter
// The iterator methods are inlined and optimized

// Closures also use static dispatch
fn apply<F: Fn(i32) -> i32>(f: F, val: i32) -> i32 {
    f(val) // Inlined at each call site
}

let doubled = apply(|x| x * 2, 21); // 42
```


## Resources

- [Generic Data Types](https://doc.rust-lang.org/book/ch10-01-syntax.html) — Official Rust Book chapter on generics and monomorphization

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*