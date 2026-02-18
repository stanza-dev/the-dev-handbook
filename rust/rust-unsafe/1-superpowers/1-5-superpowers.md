---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-5-superpowers"
---

# What unsafe Unlocks

`unsafe` gives you exactly 5 additional capabilities:

## 1. Dereference Raw Pointers

```rust
let x = 42;
let ptr = &x as *const i32;

unsafe {
    println!("{}", *ptr);  // Dereference raw pointer
}
```

## 2. Call Unsafe Functions

```rust
unsafe fn dangerous() {
    // ...
}

unsafe {
    dangerous();
}
```

## 3. Access Mutable Statics

```rust
static mut COUNTER: i32 = 0;

unsafe {
    COUNTER += 1;
    println!("{COUNTER}");
}
```

## 4. Implement Unsafe Traits

```rust
unsafe trait UnsafeTrait {
    fn method(&self);
}

unsafe impl UnsafeTrait for MyType {
    fn method(&self) { }
}
```

## 5. Access Union Fields

```rust
union IntOrFloat {
    i: i32,
    f: f32,
}

let u = IntOrFloat { i: 42 };
unsafe {
    println!("{}", u.f);  // Reinterpret bits as float
}
```

# What unsafe Does NOT Disable

- Borrow checker still runs!
- Type checking still happens
- Lifetime checking still works
- All other safety checks remain

`unsafe` is a promise: "I've verified this is safe."

See [Unsafe Rust](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html).

## Code Examples

**Safe wrappers around unsafe**

```rust
// unsafe block vs unsafe function

// unsafe BLOCK: "I'm doing something unsafe here"
fn safe_wrapper(ptr: *const i32) -> Option<i32> {
    if ptr.is_null() {
        return None;
    }
    // SAFETY: We checked the pointer is not null
    unsafe { Some(*ptr) }
}

// unsafe FUNCTION: "Caller must uphold invariants"
/// # Safety
/// ptr must be valid and properly aligned
unsafe fn read_ptr(ptr: *const i32) -> i32 {
    *ptr  // Caller's responsibility!
}

// Best practice: wrap unsafe in safe APIs
pub struct SafeBox {
    ptr: *mut i32,
}

impl SafeBox {
    pub fn new(value: i32) -> Self {
        let ptr = Box::into_raw(Box::new(value));
        SafeBox { ptr }
    }
    
    pub fn get(&self) -> i32 {
        // SAFETY: ptr is always valid (from Box::into_raw)
        unsafe { *self.ptr }
    }
}

impl Drop for SafeBox {
    fn drop(&mut self) {
        // SAFETY: ptr came from Box::into_raw
        unsafe { drop(Box::from_raw(self.ptr)); }
    }
}
```


## Resources

- [Unsafe Rust](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html) — The Rust Book chapter on unsafe

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*