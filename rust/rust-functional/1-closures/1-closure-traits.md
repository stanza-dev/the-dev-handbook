---
source_course: "rust-functional"
source_lesson: "rust-func-closure-traits"
---

# Fn, FnMut, FnOnce

Every closure implements one or more of these traits based on how it captures variables.

## FnOnce: Can Be Called Once

```rust
let name = String::from("Alice");

let consume = || {
    drop(name);  // Moves name out of closure
};

consume();  // OK
// consume();  // Error! Can only call once
```

## FnMut: Can Mutate Captures

```rust
let mut count = 0;

let mut increment = || {
    count += 1;  // Mutates captured variable
};

increment();  // count = 1
increment();  // count = 2
```

## Fn: Immutable Access Only

```rust
let x = 42;

let get_x = || x;  // Only reads x

println!("{}", get_x());  // 42
println!("{}", get_x());  // Can call many times
```

## Trait Hierarchy

```
FnOnce ← FnMut ← Fn
  │         │      │
  │         │      └─ Can be called infinitely (immutable access)
  │         └─ Can be called many times (mutable access)
  └─ Can be called at least once (may consume captures)
```

- `Fn` implies `FnMut` implies `FnOnce`
- A function taking `FnOnce` accepts any closure
- A function taking `Fn` requires the most restrictive closures

```rust
fn call_once<F: FnOnce()>(f: F) { f(); }
fn call_mut<F: FnMut()>(mut f: F) { f(); f(); }
fn call_fn<F: Fn()>(f: F) { f(); f(); }
```

See [Closures](https://doc.rust-lang.org/book/ch13-01-closures.html).

## Code Examples

**Closure trait examples**

```rust
// Determining closure type

// FnOnce: moves captured value
let s = String::from("hello");
let consume = move || drop(s);  // FnOnce

// FnMut: mutates captured value
let mut v = vec![1, 2];
let mutate = || v.push(3);  // FnMut

// Fn: only reads
let x = 42;
let read = || x * 2;  // Fn

// Function bounds example
fn apply_twice<F: Fn(i32) -> i32>(f: F, x: i32) -> i32 {
    f(f(x))
}

let double = |x| x * 2;
assert_eq!(apply_twice(double, 5), 20);  // 5 -> 10 -> 20

// Using closure as argument
fn process<F>(data: Vec<i32>, f: F) -> Vec<i32>
where
    F: Fn(i32) -> i32,
{
    data.into_iter().map(f).collect()
}

let result = process(vec![1, 2, 3], |x| x + 10);
assert_eq!(result, vec![11, 12, 13]);
```


## Resources

- [Closures](https://doc.rust-lang.org/book/ch13-01-closures.html) — Rust Book chapter on closures

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*