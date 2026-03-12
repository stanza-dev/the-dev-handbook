---
source_course: "rust"
source_lesson: "rust-compound-types"
---

# Compound Types: Tuples & Arrays

## Introduction

While scalar types represent single values, compound types group multiple values into one type. Rust has two primitive compound types: tuples and arrays. Both have fixed sizes known at compile time, which distinguishes them from heap-allocated collections like `Vec`. Understanding when to use each is fundamental to writing efficient Rust code.

## Key Concepts

- **Tuple**: A fixed-length collection that can hold values of different types. Accessed by index (e.g., `tup.0`) or destructuring.
- **Array**: A fixed-length collection where all elements must be the same type. Allocated on the stack for fast access.
- **Unit type**: `()` is an empty tuple that represents "no value." Functions that return nothing implicitly return `()`.
- **Vec**: A growable, heap-allocated array. Use `Vec<T>` when you do not know the size at compile time.

## Real World Context

Tuples are used constantly for returning multiple values from functions — for example, returning both a minimum and maximum from a statistics calculation. Arrays are ideal when you know the exact size at compile time, such as the months of the year or a fixed-size buffer. In performance-critical code, stack-allocated arrays avoid the heap allocation overhead of `Vec`.

## Deep Dive

Tuples group values of potentially different types with a fixed length:

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);

// Destructuring
let (x, y, z) = tup;

// Index access (zero-based)
let five_hundred = tup.0;
let six_point_four = tup.1;
```

The unit type `()` is a special empty tuple. Functions that do not return a value implicitly return it:

```rust
fn do_nothing() -> () {
    // Implicitly returns ()
}
```

Arrays have a fixed length and store elements of the same type on the stack:

```rust
let a = [1, 2, 3, 4, 5];
let a: [i32; 5] = [1, 2, 3, 4, 5]; // Explicit type and length
let a = [3; 5]; // Creates [3, 3, 3, 3, 3]

let first = a[0];
let second = a[1];
```

Rust performs runtime bounds checking on array access. Accessing an out-of-bounds index causes a panic rather than reading invalid memory:

```rust
let a = [1, 2, 3];
// a[10]; // Panics at runtime! Index out of bounds
```

When you need a dynamically sized collection, use `Vec<T>` instead:

```rust
let mut v: Vec<i32> = Vec::new();
v.push(1);
v.push(2);
// or use the vec! macro
let v = vec![1, 2, 3];
```

The key differences between arrays and vectors:

| Feature | Array | Vec |
|---------|-------|-----|
| Size | Fixed at compile time | Dynamic |
| Storage | Stack | Heap |
| Use case | Known, small size | Unknown or growing |

## Common Pitfalls

1. **Using arrays when size is unknown** — If the number of elements depends on runtime input (user data, file contents), you must use `Vec<T>`. Arrays require a compile-time constant for their length.
2. **Forgetting that tuple access uses `.` not `[]`** — Tuple elements are accessed with dot syntax (`tup.0`), not bracket syntax (`tup[0]`). This is because each position can have a different type.
3. **Ignoring bounds-check panics** — While Rust prevents memory corruption, out-of-bounds access still crashes your program. Use `.get()` for safe access that returns `Option<&T>`.

## Best Practices

1. **Prefer tuples for returning multiple values** — Instead of creating a struct for one-off grouped returns, use a tuple: `fn min_max(data: &[i32]) -> (i32, i32)`.
2. **Use arrays for fixed, known collections** — Months of the year, days of the week, or lookup tables are natural fits for arrays. Use `Vec` for everything else.

## Summary

- Tuples hold values of different types with a fixed length; access with `.0`, `.1` or destructuring.
- The unit type `()` represents no value and is returned implicitly by functions.
- Arrays are fixed-size, same-type collections allocated on the stack.
- Use `Vec<T>` when you need a dynamically sized collection.
- Rust bounds-checks array access at runtime, panicking instead of allowing invalid memory reads.

## Code Examples

**Tuples for multiple return values**

```rust
fn main() {
    // Tuple for returning multiple values
    fn calculate_stats(numbers: &[i32]) -> (i32, i32, f64) {
        let min = *numbers.iter().min().unwrap();
        let max = *numbers.iter().max().unwrap();
        let avg = numbers.iter().sum::<i32>() as f64 / numbers.len() as f64;
        (min, max, avg)
    }
    
    let (min, max, avg) = calculate_stats(&[1, 2, 3, 4, 5]);
    println!("Min: {min}, Max: {max}, Avg: {avg}");
}
```


## Resources

- [Compound Types](https://doc.rust-lang.org/stable/book/ch03-02-data-types.html#compound-types) — Official Rust Book section on tuples and arrays
- [Vec - Rust Standard Library](https://doc.rust-lang.org/std/vec/struct.Vec.html) — API reference for the Vec type

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*