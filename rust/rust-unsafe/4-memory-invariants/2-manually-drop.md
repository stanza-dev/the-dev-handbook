---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-manually-drop"
---

# ManuallyDrop: Control Destructor Timing

```rust
use std::mem::ManuallyDrop;

let mut x = ManuallyDrop::new(String::from("hello"));

// Use x normally
println!("{}", *x);

// Explicitly drop when ready
unsafe {
    ManuallyDrop::drop(&mut x);
}
// Don't use x after this!
```

## Use Cases

### 1. Union with Drop Types

```rust
union MyUnion {
    s: ManuallyDrop<String>,
    n: i32,
}

let mut u = MyUnion { s: ManuallyDrop::new(String::from("hi")) };

// Must manually drop before changing
unsafe {
    ManuallyDrop::drop(&mut u.s);
    u.n = 42;
}
```

### 2. FFI Ownership Transfer

```rust
// Give ownership to C without running destructor
fn give_to_c(data: Vec<u8>) -> *mut u8 {
    let mut data = ManuallyDrop::new(data);
    data.as_mut_ptr()
    // Vec's drop doesn't run - C owns the memory now
}
```

### 3. Implementing Drop Manually

```rust
struct MyVec<T> {
    ptr: *mut T,
    len: usize,
    cap: usize,
}

impl<T> Drop for MyVec<T> {
    fn drop(&mut self) {
        // Drop all elements
        for i in 0..self.len {
            unsafe {
                std::ptr::drop_in_place(self.ptr.add(i));
            }
        }
        // Deallocate memory
        // ...
    }
}
```

## mem::forget

```rust
let s = String::from("hello");
std::mem::forget(s);  // Don't run destructor
// Memory is leaked!
```

`forget` is safe (leaking isn't UB) but usually a bug.

## Code Examples

**ManuallyDrop for self-referential types**

```rust
use std::mem::ManuallyDrop;

// Self-referential struct (simplified)
struct SelfRef {
    data: String,
    ptr: *const String,
}

impl SelfRef {
    fn new(s: &str) -> ManuallyDrop<Box<Self>> {
        let mut boxed = Box::new(SelfRef {
            data: s.to_string(),
            ptr: std::ptr::null(),
        });
        
        // Point to self
        boxed.ptr = &boxed.data;
        
        // Prevent automatic drop (would invalidate ptr)
        ManuallyDrop::new(boxed)
    }
    
    fn get_data(&self) -> &str {
        // SAFETY: ptr is always valid while self exists
        unsafe { &*self.ptr }
    }
}

impl Drop for SelfRef {
    fn drop(&mut self) {
        println!("Dropping SelfRef");
    }
}

// Usage:
let sr = SelfRef::new("hello");
println!("{}", sr.get_data());
// Must manually drop:
// unsafe { ManuallyDrop::drop(&mut sr); }
```


---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*