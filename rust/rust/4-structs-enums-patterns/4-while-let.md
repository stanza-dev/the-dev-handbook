---
source_course: "rust"
source_lesson: "rust-if-let-while-let"
---

# Concise Control Flow with if let

When you only care about one pattern:

```rust
// Verbose match
match config_max {
    Some(max) => println!("Max is {max}"),
    _ => (),
}

// Concise if let
if let Some(max) = config_max {
    println!("Max is {max}");
}

// With else
if let Some(max) = config_max {
    println!("Max is {max}");
} else {
    println!("No max configured");
}
```

## while let

Loop while pattern matches:

```rust
let mut stack = vec![1, 2, 3];

while let Some(top) = stack.pop() {
    println!("{top}");
}
// Prints: 3, 2, 1
```

## let else (Rust 1.65+)

The refutable pattern version of `let`:

```rust
fn get_count_item(s: &str) -> (u64, &str) {
    let mut it = s.split(' ');
    
    let Some(count_str) = it.next() else {
        panic!("No count!");
    };
    let Some(item) = it.next() else {
        panic!("No item!");
    };
    
    let Ok(count) = count_str.parse::<u64>() else {
        panic!("Invalid count!");
    };
    
    (count, item)
}
```

See [Concise Control Flow with if let](https://doc.rust-lang.org/stable/book/ch06-03-if-let.html).

## Code Examples

**if let patterns**

```rust
// Processing Option chains
let numbers = vec![Some(1), None, Some(3)];

for num in numbers {
    if let Some(n) = num {
        println!("Found: {n}");
    }
}

// With Result
if let Ok(value) = "42".parse::<i32>() {
    println!("Parsed: {value}");
}
```


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*