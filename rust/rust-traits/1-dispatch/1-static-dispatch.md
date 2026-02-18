---
source_course: "rust-traits"
source_lesson: "rust-traits-static-dispatch"
---

# Generics = Static Dispatch

When you use generics, Rust generates specialized code for each concrete type:

```rust
fn print_it<T: Display>(item: T) {
    println!("{item}");
}

print_it(42);        // Generates print_it::<i32>
print_it("hello");   // Generates print_it::<&str>
print_it(3.14_f64);  // Generates print_it::<f64>
```

## Monomorphization

The compiler "monomorphizes" generics - creating concrete implementations:

```rust
// You write:
fn largest<T: Ord>(list: &[T]) -> &T { ... }

// Compiler generates (conceptually):
fn largest_i32(list: &[i32]) -> &i32 { ... }
fn largest_f64(list: &[f64]) -> &f64 { ... }
fn largest_String(list: &[String]) -> &String { ... }
```

## Benefits of Static Dispatch

1. **Zero overhead**: No vtable lookup
2. **Inlining**: The compiler can inline the concrete implementation
3. **Optimization**: Type-specific optimizations possible

## Downsides

1. **Code bloat**: Each instantiation generates code
2. **Longer compile times**: More code to generate and optimize
3. **No heterogeneous collections**: Can't mix types

```rust
// This won't work:
let printables: Vec<T: Display> = vec![42, "hello"]; // Error!
```

See [Generics](https://doc.rust-lang.org/book/ch10-01-syntax.html).

## Code Examples

**Static dispatch optimization**

```rust
// Static dispatch allows inlining
fn process<F: Fn(i32) -> i32>(f: F, x: i32) -> i32 {
    f(x)  // Can be inlined!
}

let double = |x| x * 2;
let result = process(double, 21);
// Compiler can inline double directly

// Trait bounds are zero-cost
fn sum<I: Iterator<Item = i32>>(iter: I) -> i32 {
    iter.fold(0, |acc, x| acc + x)
}

let total = sum(vec![1, 2, 3].into_iter());
// Specialized to Vec's iterator, fully optimized
```


---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*