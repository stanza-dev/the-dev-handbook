---
source_course: "rust"
source_lesson: "rust-vectors"
---

# Vec<T>: Growable Arrays

Vectors are resizable arrays stored on the heap.

```rust
// Creating vectors
let v: Vec<i32> = Vec::new();
let v = vec![1, 2, 3]; // Macro shorthand

// Growing a vector
let mut v = Vec::new();
v.push(5);
v.push(6);
v.push(7);

// Accessing elements
let third = &v[2];           // Panics if out of bounds
let third = v.get(2);        // Returns Option<&T>
```

## Safe Access with get()

```rust
let v = vec![1, 2, 3, 4, 5];

match v.get(100) {
    Some(value) => println!("Got {value}"),
    None => println!("Index out of bounds"),
}
```

## Iterating

```rust
let v = vec![100, 32, 57];

// Immutable iteration
for i in &v {
    println!("{i}");
}

// Mutable iteration
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50; // Dereference to modify
}
```

## Useful Methods

```rust
let mut v = vec![1, 2, 3];

v.push(4);            // Add to end
v.pop();              // Remove from end -> Option<T>
v.insert(0, 0);       // Insert at index
v.remove(0);          // Remove at index
v.len();              // Length
v.is_empty();         // Check if empty
v.contains(&2);       // Check membership
v.sort();             // Sort in place
v.dedup();            // Remove consecutive duplicates
```

See [Storing Lists of Values with Vectors](https://doc.rust-lang.org/stable/book/ch08-01-vectors.html).

## Code Examples

**Heterogeneous vectors via enum**

```rust
// Store different types with enum
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*