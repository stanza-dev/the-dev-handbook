---
source_course: "rust"
source_lesson: "rust-slices"
---

# Slices: References to Contiguous Sequences

Slices let you reference a contiguous sequence of elements rather than the whole collection.

```rust
let s = String::from("hello world");

let hello = &s[0..5];   // "hello"
let world = &s[6..11];  // "world"

// Shorthand
let hello = &s[..5];    // Start from 0
let world = &s[6..];    // Go to end
let whole = &s[..];     // Whole string
```

## String Slices are `&str`

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    
    &s[..]
}

// Works with both String and &str!
let s = String::from("hello world");
let word = first_word(&s);
let word = first_word("hello world"); // String literal
```

## Array Slices

```rust
let a = [1, 2, 3, 4, 5];
let slice: &[i32] = &a[1..3]; // [2, 3]
assert_eq!(slice, &[2, 3]);
```

## Slices Prevent Bugs

```rust
let mut s = String::from("hello world");
let word = first_word(&s);
// s.clear(); // Error! Can't mutate while word (immutable ref) exists
println!("the first word is: {}", word);
```

See [The Slice Type](https://doc.rust-lang.org/stable/book/ch04-03-slices.html).

## Code Examples

**Flexible function signatures with &str**

```rust
fn main() {
    // Prefer &str over &String in function parameters
    fn print_length(s: &str) {
        println!("Length: {}", s.len());
    }
    
    let owned = String::from("hello");
    let literal = "world";
    
    print_length(&owned);  // Works with String
    print_length(literal); // Works with &str
    print_length(&owned[1..4]); // Works with slices
}
```


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*