---
source_course: "rust-ownership"
source_lesson: "rust-ownership-lifetime-syntax"
---

# Lifetime Annotation Syntax

Lifetime parameters start with `'` (e.g., `'a`, `'b`).

## Function Signatures

```rust
// Both inputs and output share the same lifetime
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

This says: "The returned reference will live at least as long as the *shortest* of x and y."

## Multiple Lifetimes

```rust
// Different lifetimes when needed
fn first_word<'a, 'b>(s: &'a str, _other: &'b str) -> &'a str {
    // Return only depends on 's'
    s.split_whitespace().next().unwrap_or(s)
}
```

## Lifetime Elision Rules

The compiler applies these rules automatically:

1. **Each input reference gets its own lifetime**
   ```rust
   fn foo(x: &i32, y: &i32)  // becomes
   fn foo<'a, 'b>(x: &'a i32, y: &'b i32)
   ```

2. **If exactly one input lifetime, it's assigned to all outputs**
   ```rust
   fn foo(x: &i32) -> &i32  // becomes
   fn foo<'a>(x: &'a i32) -> &'a i32
   ```

3. **If there's `&self` or `&mut self`, its lifetime is assigned to outputs**
   ```rust
   impl Foo {
       fn method(&self) -> &str  // becomes
       fn method<'a>(&'a self) -> &'a str
   }
   ```

See [Lifetime Elision](https://doc.rust-lang.org/reference/lifetime-elision.html).

## Code Examples

**Struct and impl with lifetimes**

```rust
// Struct with lifetime
struct Excerpt<'a> {
    part: &'a str,
}

impl<'a> Excerpt<'a> {
    // Elision rule 3: output gets lifetime of &self
    fn get(&self) -> &str {
        self.part
    }
    
    // Explicit when returning something else
    fn with_prefix(&self, prefix: &str) -> String {
        format!("{}{}", prefix, self.part)
    }
}
```


---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*