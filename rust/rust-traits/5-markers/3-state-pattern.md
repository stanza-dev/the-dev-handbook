---
source_course: "rust-traits"
source_lesson: "rust-traits-type-state-pattern"
---

# The Type-State Pattern

## Introduction
The type-state pattern uses marker types and `PhantomData` to encode state machines in the type system. Invalid state transitions become compile-time errors, eliminating entire categories of runtime bugs.

## Key Concepts
- **Type-State**: Encoding an object's state as a type parameter, so the compiler enforces valid transitions.
- **State Marker**: An empty struct representing a state (e.g., `struct Open;`, `struct Closed;`).
- **Consuming Transition**: Methods that take `self` by value and return a new type with a different state parameter.

## Real World Context
HTTP client builders use type-state to ensure required fields are set before building. Database transaction types use it to prevent operations after commit/rollback. Protocol implementations use it to enforce valid message ordering.

## Deep Dive

### Basic State Machine

```rust
use std::marker::PhantomData;

struct Open;
struct Closed;
struct Locked;

struct Door<State> {
    _state: PhantomData<State>,
}

impl Door<Closed> {
    fn new() -> Self { Door { _state: PhantomData } }
    fn open(self) -> Door<Open> { Door { _state: PhantomData } }
    fn lock(self) -> Door<Locked> { Door { _state: PhantomData } }
}

impl Door<Open> {
    fn close(self) -> Door<Closed> { Door { _state: PhantomData } }
    // Cannot lock an open door — method not implemented!
}

impl Door<Locked> {
    fn unlock(self) -> Door<Closed> { Door { _state: PhantomData } }
}
```

Invalid transitions are compile-time errors:

```rust
let door = Door::<Closed>::new();
let door = door.open();   // Door<Open>
let door = door.close();  // Door<Closed>
let door = door.lock();   // Door<Locked>
// door.open();  // Error! Door<Locked> has no .open() method
```

### Builder Pattern with Type-State

```rust
struct NoUrl;
struct HasUrl;

struct RequestBuilder<U> {
    url: Option<String>,
    headers: Vec<(String, String)>,
    _state: PhantomData<U>,
}

impl RequestBuilder<NoUrl> {
    fn new() -> Self {
        RequestBuilder { url: None, headers: vec![], _state: PhantomData }
    }
    fn url(self, url: &str) -> RequestBuilder<HasUrl> {
        RequestBuilder { url: Some(url.to_string()), headers: self.headers, _state: PhantomData }
    }
}

impl RequestBuilder<HasUrl> {
    fn send(self) -> Result<(), String> {
        // Only callable after url() has been set
        Ok(())
    }
}

// RequestBuilder::new().send(); // Error! send() only exists on HasUrl
RequestBuilder::new().url("https://example.com").send().unwrap(); // ✓
```

## Common Pitfalls
1. **Too many states** — A state machine with 10+ states creates a combinatorial explosion of impl blocks. Keep state machines small or use enums for complex cases.
2. **Forgetting that transitions consume self** — Each transition takes `self` by value, invalidating the previous state. This is intentional but surprising to newcomers.

## Best Practices
1. **Use type-state for critical invariants** — Apply it where invalid state transitions would cause security issues or data corruption.
2. **Combine with the builder pattern** — Type-state builders ensure required fields are set before calling `.build()` or `.send()`.

## Summary
- The type-state pattern encodes states as type parameters using marker types and `PhantomData`.
- Invalid transitions become compile-time errors.
- Methods consume `self` and return a new type with the next state.
- Ideal for builders, protocol state machines, and security-critical APIs.

## Code Examples

**A type-state request builder that enforces at compile time that a URL must be set before the request can be sent**

```rust
use std::marker::PhantomData;

// Type-state builder: ensures url is set before send
struct NoUrl;
struct HasUrl;

struct Request<S> {
    url: Option<String>,
    _state: PhantomData<S>,
}

impl Request<NoUrl> {
    fn new() -> Self {
        Request { url: None, _state: PhantomData }
    }
    fn url(self, url: &str) -> Request<HasUrl> {
        Request { url: Some(url.to_string()), _state: PhantomData }
    }
}

impl Request<HasUrl> {
    fn send(self) { println!("Sending to {}", self.url.unwrap()); }
}

// Request::new().send(); // Compile error!
Request::new().url("https://example.com").send(); // OK
```


## Resources

- [Type-State Pattern](https://doc.rust-lang.org/std/marker/struct.PhantomData.html) — PhantomData documentation — the core tool for implementing type-state patterns

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*