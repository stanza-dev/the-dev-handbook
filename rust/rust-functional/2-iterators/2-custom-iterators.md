---
source_course: "rust-functional"
source_lesson: "rust-func-custom-iterators"
---

# The Iterator Trait

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // 70+ methods provided for free!
}
```

## Simple Counter

```rust
struct Counter {
    current: u32,
    max: u32,
}

impl Counter {
    fn new(max: u32) -> Self {
        Counter { current: 0, max }
    }
}

impl Iterator for Counter {
    type Item = u32;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.current < self.max {
            self.current += 1;
            Some(self.current)
        } else {
            None
        }
    }
}

let counter = Counter::new(5);
let sum: u32 = counter.sum();  // 1 + 2 + 3 + 4 + 5 = 15
```

## Iterator over References

```rust
struct Windows<'a, T> {
    data: &'a [T],
    size: usize,
    pos: usize,
}

impl<'a, T> Iterator for Windows<'a, T> {
    type Item = &'a [T];
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.pos + self.size <= self.data.len() {
            let window = &self.data[self.pos..self.pos + self.size];
            self.pos += 1;
            Some(window)
        } else {
            None
        }
    }
}
```

## IntoIterator

```rust
struct MyCollection {
    items: Vec<i32>,
}

impl IntoIterator for MyCollection {
    type Item = i32;
    type IntoIter = std::vec::IntoIter<i32>;
    
    fn into_iter(self) -> Self::IntoIter {
        self.items.into_iter()
    }
}

// Now you can use for loops
let col = MyCollection { items: vec![1, 2, 3] };
for item in col {
    println!("{item}");
}
```

## Code Examples

**Fibonacci iterator**

```rust
// Fibonacci iterator
struct Fibonacci {
    a: u64,
    b: u64,
}

impl Fibonacci {
    fn new() -> Self {
        Fibonacci { a: 0, b: 1 }
    }
}

impl Iterator for Fibonacci {
    type Item = u64;
    
    fn next(&mut self) -> Option<Self::Item> {
        let next = self.a;
        self.a = self.b;
        self.b = next.saturating_add(self.a);
        Some(next)
    }
}

// Get first 10 Fibonacci numbers
let fibs: Vec<_> = Fibonacci::new().take(10).collect();
assert_eq!(fibs, vec![0, 1, 1, 2, 3, 5, 8, 13, 21, 34]);

// Use all iterator methods for free!
let sum: u64 = Fibonacci::new().take(20).sum();
let evens: Vec<_> = Fibonacci::new()
    .filter(|&x| x % 2 == 0)
    .take(5)
    .collect();
```


---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*