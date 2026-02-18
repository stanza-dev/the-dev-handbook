---
source_course: "rust"
source_lesson: "rust-borrowing-references"
---

# References: Borrowing Without Ownership

Instead of taking ownership, you can **borrow** a value using references (`&`).

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // Pass a reference
    println!("The length of '{}' is {}.", s1, len); // s1 still valid!
}

fn calculate_length(s: &String) -> usize {
    s.len()
} // s goes out of scope, but doesn't own the data, so nothing is dropped
```

## The Borrowing Rules

**At any given time, you can have either:**
- **One mutable reference**, OR
- **Any number of immutable references**

**Never both at the same time!**

```rust
let mut s = String::from("hello");

let r1 = &s;     // OK
let r2 = &s;     // OK - multiple immutable refs
// let r3 = &mut s; // Error! Can't borrow as mutable while immutable refs exist

println!("{} and {}", r1, r2);
// r1 and r2 no longer used after this point

let r3 = &mut s; // OK now - no immutable refs in scope
```

## Mutable References

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
    println!("{}", s); // "hello, world"
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

## Why These Rules?

They prevent **data races** at compile time. A data race occurs when:
- Two or more pointers access the same data simultaneously
- At least one is writing
- There's no synchronization

Rust makes data races impossible!

See [References and Borrowing](https://doc.rust-lang.org/stable/book/ch04-02-references-and-borrowing.html).

## Code Examples

**Preventing iterator invalidation**

```rust
fn main() {
    let mut v = vec![1, 2, 3];
    
    // This would be a data race in C++!
    // let first = &v[0];  // Immutable borrow
    // v.push(4);          // Mutable borrow - Error!
    // println!("{}", first);
    
    // Correct approach:
    v.push(4);
    let first = &v[0];
    println!("First: {}", first);
}
```


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*