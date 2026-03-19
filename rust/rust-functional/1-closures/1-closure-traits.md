---
source_course: "rust-functional"
source_lesson: "rust-func-closure-traits"
---

# The Three Closure Traits

## Introduction
Every closure in Rust implements one or more of three traits — `Fn`, `FnMut`, and `FnOnce` — determined by how the closure captures and uses variables from its environment. Understanding this hierarchy is the foundation for writing generic APIs that accept closures.

## Key Concepts
- **FnOnce**: A closure that can be called at least once. It may consume (move) captured variables, so calling it a second time is not guaranteed to be safe.
- **FnMut**: A closure that can be called multiple times and may mutate its captured state. It borrows captured variables mutably.
- **Fn**: A closure that can be called any number of times without side effects on its captures. It borrows captured variables immutably (or captures nothing).
- **Trait hierarchy**: `Fn` is a subtrait of `FnMut`, which is a subtrait of `FnOnce`. Any `Fn` closure satisfies `FnMut`, and any `FnMut` closure satisfies `FnOnce`.

## Real World Context
Every time you pass a closure to `.map()`, `.filter()`, `thread::spawn()`, or any generic callback API, the compiler decides which trait that closure implements. Understanding these traits lets you write functions that accept the widest or narrowest set of closures appropriate for your use case.

## Deep Dive

The compiler determines a closure's trait based on what the closure body does with captured variables. Let's walk through each trait with clear examples.

### FnOnce — consumes captured values

A closure that moves a captured value out of itself can only be called once, because the value is gone after the first call:

```rust
let name = String::from("Alice");

let consume = || {
    drop(name); // Moves `name` out, consuming it
};

consume();  // OK — first call
// consume();  // Error! `name` was consumed on the first call
```

After `consume()` runs, `name` no longer exists inside the closure. The compiler marks this closure as `FnOnce` only.

### FnMut — mutates captured values

A closure that changes a captured variable needs mutable access, so it implements `FnMut`:

```rust
let mut count = 0;

let mut increment = || {
    count += 1; // Mutably borrows `count`
};

increment(); // count = 1
increment(); // count = 2
```

Notice that both the binding `count` and the closure binding `increment` must be declared `mut`. This closure also satisfies `FnOnce` (it can certainly be called once), but it does not satisfy `Fn` because it mutates state.

### Fn — immutable access only

A closure that only reads captured variables (or captures nothing) implements `Fn`:

```rust
let greeting = String::from("Hello");

let greet = || println!("{greeting}"); // Immutably borrows `greeting`

greet(); // "Hello"
greet(); // Can call as many times as needed
```

This closure satisfies all three traits: `Fn`, `FnMut`, and `FnOnce`.

### Writing generic functions with closure bounds

When you write a function that takes a closure, choose the least restrictive bound that works:

```rust
// Accepts any closure (even consuming ones)
fn call_once<F: FnOnce() -> String>(f: F) -> String { f() }

// Accepts closures that can be called multiple times (may mutate)
fn call_twice<F: FnMut() -> i32>(mut f: F) -> i32 { f() + f() }

// Accepts only pure closures (no mutation)
fn call_many<F: Fn() -> i32>(f: F) -> i32 { f() + f() + f() }
```

Using `FnOnce` as a bound is the most flexible because it accepts all closures. Using `Fn` is the most restrictive.

## Common Pitfalls
1. **Requiring `Fn` when `FnOnce` suffices** — If you only call the closure once, use `FnOnce` as the bound. Using `Fn` unnecessarily rejects valid closures that consume their captures.
2. **Forgetting `mut` on `FnMut` closures** — Both the closure variable and the parameter in a generic function must be marked `mut` when calling an `FnMut` closure.
3. **Confusing the hierarchy direction** — `Fn` is the *most* restrictive trait (fewest closures qualify), not the least. Think of it as: `Fn` ⊂ `FnMut` ⊂ `FnOnce`.

## Best Practices
1. **Default to `FnOnce` for single-call callbacks** — This accepts the widest range of closures. Tighten to `FnMut` or `Fn` only when you need to call the closure multiple times.
2. **Use `impl Fn(...)` in argument position** — For most application code, `impl Fn(i32) -> i32` is cleaner than a named generic parameter.
3. **Annotate return types with `impl Fn`** — When returning closures, use `-> impl Fn(...)` and add `move` if the closure captures local variables.

## Summary
- Closures implement `FnOnce`, `FnMut`, or `Fn` based on how they use captures.
- The trait hierarchy is `Fn` ⊂ `FnMut` ⊂ `FnOnce`.
- Choose the least restrictive bound (`FnOnce`) unless you need repeated calls.
- `Fn` closures can be shared freely; `FnMut` closures require `mut` access.
- Rust 1.94 makes closure capturing more precise around pattern bindings.

## Code Examples

**Three functions demonstrating Fn, FnMut, and FnOnce bounds — each accepts a different level of closure capability**

```rust
// Demonstrating the trait hierarchy with a practical example

fn apply_twice<F: Fn(i32) -> i32>(f: F, value: i32) -> i32 {
    f(f(value))
}

fn apply_and_collect<F: FnMut(i32) -> i32>(mut f: F, items: Vec<i32>) -> Vec<i32> {
    items.into_iter().map(|x| f(x)).collect()
}

fn consume_and_report<F: FnOnce() -> String>(f: F) {
    println!("Result: {}", f());
}

fn main() {
    // Fn closure — no mutation, no consumption
    let multiplier = 3;
    let triple = |x| x * multiplier;
    let result = apply_twice(triple, 5); // 5 -> 15 -> 45
    println!("apply_twice: {result}");

    // FnMut closure — tracks call count
    let mut call_count = 0;
    let counting_double = |x| {
        call_count += 1;
        x * 2
    };
    let doubled = apply_and_collect(counting_double, vec![1, 2, 3]);
    // doubled = [2, 4, 6], call_count = 3

    // FnOnce closure — consumes a String
    let message = String::from("task complete");
    consume_and_report(|| message); // `message` is moved
}
```


## Resources

- [Closures: Anonymous Functions that Capture Their Environment](https://doc.rust-lang.org/book/ch13-01-closures.html) — Official Rust Book chapter covering closure syntax, capturing, and the Fn traits

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*