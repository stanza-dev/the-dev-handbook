---
source_course: "rust"
source_lesson: "rust-compound-types"
---

# Compound Types

Compound types group multiple values into one type.

## Tuples

Fixed length, heterogeneous types:

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);

// Destructuring
let (x, y, z) = tup;

// Index access (zero-based)
let five_hundred = tup.0;
let six_point_four = tup.1;
```

**The Unit Type:** `()` is an empty tuple, used when no value is returned.

```rust
fn do_nothing() -> () {
    // Implicitly returns ()
}
```

## Arrays

Fixed length, homogeneous types, allocated on the **stack**:

```rust
let a = [1, 2, 3, 4, 5];
let a: [i32; 5] = [1, 2, 3, 4, 5]; // Explicit type
let a = [3; 5]; // [3, 3, 3, 3, 3]

let first = a[0];
let second = a[1];
```

**Runtime bounds checking:**
```rust
let a = [1, 2, 3];
// a[10]; // Panics at runtime! Index out of bounds
```

## Arrays vs Vectors

| Feature | Array | Vec |
|---------|-------|-----|
| Size | Fixed at compile time | Dynamic |
| Storage | Stack | Heap |
| Use case | Known size | Unknown/growing |

```rust
// Use Vec when size is dynamic
let mut v: Vec<i32> = Vec::new();
v.push(1);
v.push(2);
// or
let v = vec![1, 2, 3];
```

See [Compound Types](https://doc.rust-lang.org/stable/book/ch03-02-data-types.html#compound-types).

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


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*