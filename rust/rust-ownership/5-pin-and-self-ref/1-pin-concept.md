---
source_course: "rust-ownership"
source_lesson: "rust-ownership-pin-concept"
---

# Self-Referential Structs

If a struct contains a pointer to itself, moving it breaks the pointer:

```rust
struct SelfRef {
    value: String,
    ptr: *const String,  // Points to value above
}

let mut s = SelfRef {
    value: String::from("hello"),
    ptr: std::ptr::null(),
};
s.ptr = &s.value;  // ptr now points to value

// If we move s...
let s2 = s;  // value moves, but ptr still points to old location!
// s2.ptr is now dangling!
```

## Why This Matters

Async functions create self-referential state machines:

```rust
async fn example() {
    let data = vec![1, 2, 3];
    let reference = &data[0];  // Self-reference!
    some_async_op().await;     // State machine stores both
    println!("{reference}");   // reference points within same struct
}
```

## The Solution: Pin<P>

`Pin<P>` guarantees the pointee won't move:

```rust
use std::pin::Pin;
use std::marker::PhantomPinned;

struct Unmovable {
    data: String,
    _pin: PhantomPinned,  // Makes type !Unpin
}

let pinned: Pin<Box<Unmovable>> = Box::pin(Unmovable {
    data: String::from("hello"),
    _pin: PhantomPinned,
});

// Can't move out of pinned!
// let moved = *pinned;  // Error!
```

See [Pin Documentation](https://doc.rust-lang.org/std/pin/index.html).

## Code Examples

**Self-referential struct with Pin**

```rust
use std::pin::Pin;
use std::marker::PhantomPinned;

struct SelfReferential {
    data: String,
    slice: *const str,  // Will point into data
    _pin: PhantomPinned,
}

impl SelfReferential {
    fn new(data: String) -> Pin<Box<Self>> {
        let res = Self {
            data,
            slice: std::ptr::null(),
            _pin: PhantomPinned,
        };
        let mut boxed = Box::pin(res);
        
        // Safe: we're setting up the self-reference
        // before anyone can observe it
        let slice: *const str = &boxed.data;
        unsafe {
            let mut_ref = Pin::as_mut(&mut boxed);
            Pin::get_unchecked_mut(mut_ref).slice = slice;
        }
        
        boxed
    }
    
    fn get_slice(self: Pin<&Self>) -> &str {
        unsafe { &*self.slice }
    }
}
```


## Resources

- [Pin and Unpin](https://doc.rust-lang.org/std/pin/index.html) — Official Pin documentation

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*