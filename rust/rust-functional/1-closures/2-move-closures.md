---
source_course: "rust-functional"
source_lesson: "rust-func-move-closures"
---

# Default Capturing

By default, closures borrow by the least restrictive method:

```rust
let s = String::from("hello");

let print_s = || println!("{s}");  // Borrows &s

print_s();
println!("{s}");  // Still valid!
```

# The move Keyword

Force ownership transfer into the closure:

```rust
let s = String::from("hello");

let consume_s = move || println!("{s}");  // Owns s

consume_s();
// println!("{s}");  // Error! s was moved
```

## When to Use move

### 1. Spawning Threads

```rust
use std::thread;

let data = vec![1, 2, 3];

let handle = thread::spawn(move || {
    println!("{:?}", data);  // data moved into thread
});

handle.join().unwrap();
```

### 2. Returning Closures

```rust
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n  // n must be moved in
}

let add_5 = make_adder(5);
assert_eq!(add_5(10), 15);
```

### 3. Async Blocks

```rust
async fn fetch_data(url: String) {
    let data = async move {
        // url moved into async block
        fetch(&url).await
    };
}
```

## move with Copy Types

```rust
let x = 42;  // i32 is Copy

let closure = move || x;  // x is copied, not moved

println!("{x}");  // Still valid!
```

## Code Examples

**Move closure patterns**

```rust
use std::thread;

// Move is essential for threads
fn parallel_sum(data: Vec<i32>) -> i32 {
    let mid = data.len() / 2;
    let (left, right) = data.split_at(mid);
    
    // Clone for the spawned thread
    let left = left.to_vec();
    let right = right.to_vec();
    
    let left_handle = thread::spawn(move || {
        left.iter().sum::<i32>()
    });
    
    let right_sum: i32 = right.iter().sum();
    let left_sum = left_handle.join().unwrap();
    
    left_sum + right_sum
}

// Closure capturing multiple values
fn capture_example() {
    let a = String::from("hello");
    let b = vec![1, 2, 3];
    let c = 42;
    
    // move takes ownership of a, b
    // c is copied (it's Copy)
    let closure = move || {
        println!("{a}, {:?}, {c}", b);
    };
    
    closure();
    // a, b are gone
    println!("{c}");  // c still works (was copied)
}
```


---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*