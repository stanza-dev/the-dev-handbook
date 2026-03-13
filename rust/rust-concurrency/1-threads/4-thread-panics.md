---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-thread-panics"
---

# Thread Panics & Error Handling

## Introduction

Threads can panic, and when they do, the panic does not propagate to the parent thread automatically. Instead, Rust captures the panic payload inside the `JoinHandle`. Understanding how to detect, handle, and recover from thread panics is critical for building robust concurrent applications that do not silently lose work.

## Key Concepts

- **Panic Isolation**: A panic in a spawned thread does not crash the entire process. The panic is captured and stored in the `JoinHandle`, which returns `Err` when joined.
- **JoinHandle::join()**: Returns `Result<T, Box<dyn Any + Send>>`. `Ok(value)` on success, `Err(payload)` if the thread panicked. The payload is the panic message or value.
- **std::panic::catch_unwind**: Catches panics within the current thread, similar to try/catch in other languages. Returns `Result<T, Box<dyn Any + Send>>`.
- **Panic Hooks**: `std::panic::set_hook` lets you install a global handler that runs when any thread panics, useful for logging or crash reporting.

## Real World Context

In a web server with a thread pool, a panic in one request handler should not take down the entire server. By catching panics via `JoinHandle::join()` or `catch_unwind`, the server can log the error, return a 500 response, and continue serving other requests. Build systems and test runners also rely on panic isolation to report failures without aborting the entire run.

## Deep Dive

When a spawned thread panics, `join()` returns an `Err` containing the panic payload. You can downcast this payload to extract the panic message.

```rust
use std::thread;

let handle = thread::spawn(|| {
    panic!("something went wrong!");
});

match handle.join() {
    Ok(value) => println!("thread returned: {value:?}"),
    Err(panic_payload) => {
        if let Some(msg) = panic_payload.downcast_ref::<&str>() {
            println!("thread panicked with: {msg}");
        } else if let Some(msg) = panic_payload.downcast_ref::<String>() {
            println!("thread panicked with: {msg}");
        } else {
            println!("thread panicked with an unknown payload");
        }
    }
}
```

The `downcast_ref` calls attempt to interpret the panic payload as a `&str` or `String`, which covers the two most common panic message types. If neither matches, the panic used a custom type.

`std::panic::catch_unwind` lets you catch panics within the current thread without spawning a new one. The closure must be `UnwindSafe`, which most types satisfy.

```rust
use std::panic;

let result = panic::catch_unwind(|| {
    let x: i32 = "not a number".parse().unwrap(); // panics
    x
});

match result {
    Ok(value) => println!("got: {value}"),
    Err(_) => println!("caught a panic!"),
}

println!("execution continues normally after the panic");
```

This is especially useful in FFI boundaries where panics must not cross into C code, and in thread pools where you want to isolate task failures.

You can install a custom panic hook to log panics before they are captured.

```rust
use std::panic;

panic::set_hook(Box::new(|info| {
    let location = info.location().map(|l| format!("{}:{}:{}", l.file(), l.line(), l.column())).unwrap_or_default();
    eprintln!("PANIC at {location}: {info}");
}));

// Now every panic (in any thread) runs this hook before being captured
```

The hook runs before the panic is captured by `catch_unwind` or `JoinHandle::join()`, making it ideal for centralized error logging.

## Common Pitfalls

1. **Ignoring `join()` return values** — If you call `join().unwrap()` and the thread panicked, your main thread panics too. Always match on the `Result` in production code to handle thread failures gracefully.
2. **Relying on `catch_unwind` for control flow** — Panics in Rust are for unrecoverable errors. Using `catch_unwind` as a general error-handling mechanism is an anti-pattern. Prefer `Result` for expected failures.

## Best Practices

1. **Use `Result` propagation inside threads** — Have your thread closure return `Result<T, E>` and handle errors normally. Reserve `catch_unwind` for FFI boundaries and panic isolation in thread pools.
2. **Install a panic hook for observability** — In production applications, set a panic hook that logs the panic location and message to your error tracking system.

## Summary

- Thread panics are isolated: they do not crash the process, but are captured in the `JoinHandle`.
- `join()` returns `Result<T, Box<dyn Any + Send>>` — always handle the `Err` case in production.
- `catch_unwind` catches panics within the current thread, useful for FFI boundaries and thread pools.
- `set_hook` installs a global panic handler for logging and crash reporting.

## Code Examples

**Robust thread error handling with nested Result: distinguishing task errors from thread panics**

```rust
use std::thread;

fn run_task(id: usize) -> Result<String, String> {
    if id == 3 {
        return Err(format!("task {id} failed"));
    }
    Ok(format!("task {id} completed"))
}

fn main() {
    let handles: Vec<_> = (0..5)
        .map(|id| {
            thread::spawn(move || run_task(id))
        })
        .collect();

    for handle in handles {
        match handle.join() {
            Ok(Ok(msg)) => println!("Success: {msg}"),
            Ok(Err(msg)) => println!("Task error: {msg}"),
            Err(_) => println!("Thread panicked!"),
        }
    }
}
```


## Resources

- [std::thread::JoinHandle](https://doc.rust-lang.org/std/thread/struct.JoinHandle.html) — Standard library reference for JoinHandle and its join() method
- [std::panic::catch_unwind](https://doc.rust-lang.org/std/panic/fn.catch_unwind.html) — Catching panics within the current thread

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*