---
source_course: "rust"
source_lesson: "rust-strings-and-slices"
---

# Strings & String Slices

## Introduction

Strings are one of the most common sources of confusion for Rust newcomers, primarily because Rust has two main string types instead of one. This distinction exists because Rust takes UTF-8 encoding seriously and forces you to handle text correctly. Once you understand the difference between `String` and `&str`, working with text in Rust becomes straightforward.

## Key Concepts

- **String**: An owned, heap-allocated, growable UTF-8 string. You can modify, append to, and pass ownership of a `String`.
- **&str (string slice)**: A borrowed reference to a sequence of UTF-8 bytes. String literals (`"hello"`) are `&str` with a `'static` lifetime.
- **UTF-8 encoding**: Rust strings are always valid UTF-8. Characters can be 1-4 bytes, which is why indexing by byte position is dangerous.
- **Deref coercion**: A `&String` automatically coerces to `&str`, so functions that accept `&str` work with both types.

## Real World Context

Every web application, CLI tool, and API handler deals with strings constantly. In Rust, the `String` vs `&str` distinction prevents a class of bugs related to encoding, ownership, and dangling references. When you accept `&str` in function parameters, your code works with both owned strings and string literals without unnecessary allocations. Production Rust code uses this pattern everywhere.

## Deep Dive

The two main string types serve different purposes:

| Type | Description | Storage |
|------|-------------|--------|
| `String` | Owned, growable | Heap |
| `&str` | Borrowed string slice | Stack pointer + length |

You can create strings in several ways:

```rust
let s1: &str = "hello";                // String literal (static lifetime)
let s2: String = String::from("hello"); // Owned String
let s3: String = "hello".to_string();   // Same as above
```

Common string operations all work on `String`:

```rust
let mut s = String::from("foo");
s.push_str("bar");  // s = "foobar"
s.push('!');        // s = "foobar!"
```

Concatenation has ownership implications. The `+` operator takes ownership of the left operand:

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // s1 is MOVED, s2 is borrowed
// s1 is no longer valid here
```

The `format!` macro is often better because it does not take ownership of any argument:

```rust
let s = format!("{}-{}-{}", "tic", "tac", "toe");
```

A critical rule: you cannot index Rust strings with integers because UTF-8 characters vary in byte width:

```rust
let s = String::from("hello");
// let h = s[0]; // Error! Rust strings are UTF-8
```

Use byte slices carefully, or iterate over characters:

```rust
let hello = "Здравствуйте";
let s = &hello[0..4]; // First 4 BYTES (not characters!) = "Зд"

for c in "नमस्ते".chars() {
    println!("{c}");
}
```

## Common Pitfalls

1. **Slicing on non-character boundaries** — `&s[0..1]` on a multi-byte character panics at runtime. Always ensure your byte offsets land on valid UTF-8 boundaries, or use `.chars()` for safe iteration.
2. **Unnecessary `.clone()` on strings** — If a function only needs to read a string, accept `&str` instead of `String`. This avoids cloning and heap allocation.
3. **Assuming `len()` returns character count** — `String::len()` returns the number of bytes, not characters. Use `.chars().count()` for the character count.

## Best Practices

1. **Accept `&str` in function parameters** — Functions that only read string data should take `&str`, not `String` or `&String`. Thanks to deref coercion, callers can pass either type.
2. **Use `format!` for complex concatenation** — It is more readable than chaining `+` and does not transfer ownership of any argument.

## Summary

- `String` is owned and heap-allocated; `&str` is a borrowed reference to UTF-8 bytes.
- String literals are `&str` with a static lifetime.
- You cannot index strings with integers because UTF-8 characters vary in byte width.
- Use `.chars()` to iterate over characters and `&str` in function parameters for flexibility.
- The `format!` macro concatenates strings without taking ownership.

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