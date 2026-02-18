---
source_course: "rust"
source_lesson: "rust-result-error-handling"
---

# Recoverable Errors with Result

Rust doesn't have exceptions. It uses the `Result` enum.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

## Basic Usage

```rust
use std::fs::File;

let greeting_file = File::open("hello.txt");

let greeting_file = match greeting_file {
    Ok(file) => file,
    Err(error) => panic!("Problem opening file: {:?}", error),
};
```

## Matching Different Errors

```rust
use std::fs::File;
use std::io::ErrorKind;

let file = match File::open("hello.txt") {
    Ok(file) => file,
    Err(error) => match error.kind() {
        ErrorKind::NotFound => match File::create("hello.txt") {
            Ok(fc) => fc,
            Err(e) => panic!("Problem creating file: {:?}", e),
        },
        other_error => panic!("Problem opening file: {:?}", other_error),
    },
};
```

## unwrap and expect

```rust
// Panic if Err
let file = File::open("hello.txt").unwrap();

// Panic with custom message
let file = File::open("hello.txt")
    .expect("hello.txt should exist in project root");
```

## The ? Operator

Propagates errors up the call stack:

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut file = File::open("hello.txt")?; // Returns early if Err
    let mut username = String::new();
    file.read_to_string(&mut username)?;
    Ok(username)
}
```

See [Recoverable Errors with Result](https://doc.rust-lang.org/stable/book/ch09-02-recoverable-errors-with-result.html).

## Code Examples

**Error propagation**

```rust
use std::fs;

// Chaining with ?
fn read_username() -> Result<String, std::io::Error> {
    fs::read_to_string("hello.txt")
}

// In main with Result return
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let username = read_username()?;
    println!("Username: {username}");
    Ok(())
}
```


## Resources

- [Error Handling](https://doc.rust-lang.org/stable/book/ch09-00-error-handling.html) — Complete guide to error handling in Rust

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*