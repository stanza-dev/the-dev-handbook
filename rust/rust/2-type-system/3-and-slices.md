---
source_course: "rust"
source_lesson: "rust-strings-and-slices"
---

# Understanding Strings in Rust

Strings are one of the most common sources of confusion for newcomers.

## Two Main String Types

| Type | Description | Storage |
|------|-------------|--------|
| `String` | Owned, growable | Heap |
| `&str` | Borrowed string slice | Stack pointer + length |

```rust
let s1: &str = "hello";              // String literal (static lifetime)
let s2: String = String::from("hello"); // Owned String
let s3: String = "hello".to_string();   // Same as above
```

## String Operations

```rust
// Creating
let mut s = String::new();
let s = String::from("initial contents");
let s = "initial contents".to_string();

// Appending
let mut s = String::from("foo");
s.push_str("bar");  // s = "foobar"
s.push('!');        // s = "foobar!"

// Concatenation
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // s1 is MOVED, s2 is borrowed

// Format macro (doesn't take ownership)
let s = format!("{}-{}-{}", "tic", "tac", "toe");
```

## Indexing Strings

**You cannot index strings with integers!**

```rust
let s = String::from("hello");
// let h = s[0]; // Error! Rust strings are UTF-8
```

Use slices or iterators instead:

```rust
let hello = "Здравствуйте";
let s = &hello[0..4]; // First 4 BYTES (not characters!) = "Зд"

// Safe iteration
for c in "नमस्ते".chars() {
    println!("{c}");
}
```

See [Storing UTF-8 Encoded Text with Strings](https://doc.rust-lang.org/stable/book/ch08-02-strings.html).

## Code Examples

**Common string methods**

```rust
fn main() {
    // Common string operations
    let s = "  Hello, World!  ";
    
    println!("Trimmed: '{}'", s.trim());
    println!("Uppercase: {}", s.to_uppercase());
    println!("Contains 'World': {}", s.contains("World"));
    println!("Replace: {}", s.replace("World", "Rust"));
    
    // Splitting
    for word in "one,two,three".split(',') {
        println!("{word}");
    }
}
```


## Resources

- [Storing UTF-8 Text with Strings](https://doc.rust-lang.org/stable/book/ch08-02-strings.html) — Deep dive into Rust strings

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*