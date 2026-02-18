---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-type-mapping"
---

# C Type Mappings

## Primitive Types (via libc)

| C Type | Rust Type | libc alias |
|--------|-----------|------------|
| `char` | `i8` or `u8` | `c_char` |
| `short` | `i16` | `c_short` |
| `int` | `i32` | `c_int` |
| `long` | `i32`/`i64` | `c_long` |
| `long long` | `i64` | `c_longlong` |
| `size_t` | `usize` | `size_t` |
| `void*` | `*mut c_void` | - |
| `float` | `f32` | `c_float` |
| `double` | `f64` | `c_double` |

## Strings

```rust
use std::ffi::{CStr, CString};

// Rust -> C: Use CString
let rust_str = "hello";
let c_string = CString::new(rust_str).unwrap();
let c_ptr: *const i8 = c_string.as_ptr();

// C -> Rust: Use CStr
unsafe {
    let c_str = CStr::from_ptr(c_ptr);
    let rust_str: &str = c_str.to_str().unwrap();
}
```

## Structs

```rust
// C struct
// struct Point { int x; int y; };

// Rust equivalent
#[repr(C)]
struct Point {
    x: c_int,
    y: c_int,
}
```

## Pointers & Optional Values

```rust
// C: struct Foo* (nullable)
// Rust: Option<NonNull<Foo>> or *mut Foo

use std::ptr::NonNull;

extern "C" {
    // Might return null
    fn maybe_get_thing() -> *mut Thing;
}

// Safer wrapper
fn get_thing() -> Option<NonNull<Thing>> {
    unsafe { NonNull::new(maybe_get_thing()) }
}
```

## Code Examples

**Struct and string FFI**

```rust
use std::ffi::{CStr, CString};
use std::os::raw::{c_char, c_int};

// C header:
// struct Person {
//     const char* name;
//     int age;
// };
// void greet(const struct Person* p);

#[repr(C)]
struct Person {
    name: *const c_char,
    age: c_int,
}

extern "C" {
    fn greet(person: *const Person);
}

// Safe wrapper
fn greet_person(name: &str, age: i32) {
    let name_c = CString::new(name).expect("CString::new failed");
    
    let person = Person {
        name: name_c.as_ptr(),
        age: age as c_int,
    };
    
    unsafe {
        greet(&person);
    }
    // name_c is dropped here - after greet() returns
}
```


---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*