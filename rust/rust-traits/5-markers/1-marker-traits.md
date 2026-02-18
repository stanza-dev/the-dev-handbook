---
source_course: "rust-traits"
source_lesson: "rust-traits-marker-traits"
---

# Marker Traits: No Methods

Marker traits mark types with properties:

```rust
// Compiler-provided markers
trait Send {}    // Safe to transfer between threads
trait Sync {}    // Safe to share references between threads
trait Copy {}    // Bitwise copyable
trait Sized {}   // Known size at compile time
trait Unpin {}   // Safe to move after pinning
```

## Auto Traits

Some markers are automatically implemented:

```rust
// Automatically Send if all fields are Send
struct MySendType {
    x: i32,      // i32: Send ✓
    s: String,   // String: Send ✓
}  // MySendType: Send ✓

// Negative impl to opt out
impl !Send for MyType {}
```

## Custom Marker Traits

```rust
// Mark types as JSON-serializable
trait JsonSerializable {}

impl JsonSerializable for i32 {}
impl JsonSerializable for String {}
impl<T: JsonSerializable> JsonSerializable for Vec<T> {}

fn to_json<T: JsonSerializable>(value: &T) -> String {
    // Can only be called with marked types
    todo!()
}
```

## Sealed Traits

Prevent external implementations:

```rust
mod private {
    pub trait Sealed {}
}

// Only types with Sealed can implement MyTrait
pub trait MyTrait: private::Sealed {
    fn method(&self);
}

// Internal implementation
impl private::Sealed for MyType {}
impl MyTrait for MyType { ... }

// External code can't implement MyTrait
// because they can't implement Sealed
```

## Code Examples

**Type state with markers**

```rust
// Type state pattern with markers
trait DoorState {}
struct Open;
struct Closed;
struct Locked;

impl DoorState for Open {}
impl DoorState for Closed {}
impl DoorState for Locked {}

struct Door<S: DoorState> {
    _state: std::marker::PhantomData<S>,
}

impl Door<Closed> {
    fn new() -> Self {
        Door { _state: std::marker::PhantomData }
    }
    
    fn open(self) -> Door<Open> {
        Door { _state: std::marker::PhantomData }
    }
    
    fn lock(self) -> Door<Locked> {
        Door { _state: std::marker::PhantomData }
    }
}

impl Door<Open> {
    fn close(self) -> Door<Closed> {
        Door { _state: std::marker::PhantomData }
    }
    // Can't lock an open door - not implemented!
}

impl Door<Locked> {
    fn unlock(self) -> Door<Closed> {
        Door { _state: std::marker::PhantomData }
    }
}
```


---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*