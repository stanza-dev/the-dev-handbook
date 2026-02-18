---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-memory-ownership"
---

# The Golden Rule

**Memory should be freed by the same allocator that created it.**

## Returning Heap Data to C

```rust
// BAD: C might free with wrong allocator
#[no_mangle]
pub extern "C" fn get_data() -> *mut u8 {
    let data = vec![1, 2, 3];
    Box::into_raw(data.into_boxed_slice()) as *mut u8
    // Who frees this??
}

// GOOD: Provide a free function
#[no_mangle]
pub extern "C" fn create_data() -> *mut u8 {
    Box::into_raw(Box::new([1u8; 100])) as *mut u8
}

#[no_mangle]
pub extern "C" fn free_data(ptr: *mut u8) {
    if !ptr.is_null() {
        unsafe { drop(Box::from_raw(ptr as *mut [u8; 100])); }
    }
}
```

## Opaque Struct Pattern

```rust
// Rust side
pub struct Database {
    connection: String,
    // Complex internal state
}

#[no_mangle]
pub extern "C" fn db_open(conn: *const c_char) -> *mut Database {
    let conn_str = unsafe { CStr::from_ptr(conn).to_str().unwrap() };
    Box::into_raw(Box::new(Database {
        connection: conn_str.to_string(),
    }))
}

#[no_mangle]
pub extern "C" fn db_close(db: *mut Database) {
    if !db.is_null() {
        unsafe { drop(Box::from_raw(db)); }
    }
}

#[no_mangle]
pub extern "C" fn db_query(db: *mut Database, query: *const c_char) -> c_int {
    let db = unsafe { &*db };
    // Use db...
    0
}
```

```c
// C header
typedef struct Database Database;

Database* db_open(const char* conn);
void db_close(Database* db);
int db_query(Database* db, const char* query);
```

## Code Examples

**Complete opaque handle pattern**

```rust
// Safe wrapper pattern
use std::os::raw::c_char;
use std::ffi::CStr;

pub struct Config {
    values: std::collections::HashMap<String, String>,
}

// Opaque handle for C
pub type ConfigHandle = *mut Config;

#[no_mangle]
pub extern "C" fn config_new() -> ConfigHandle {
    Box::into_raw(Box::new(Config {
        values: std::collections::HashMap::new(),
    }))
}

#[no_mangle]
pub extern "C" fn config_set(
    handle: ConfigHandle,
    key: *const c_char,
    value: *const c_char,
) -> bool {
    if handle.is_null() || key.is_null() || value.is_null() {
        return false;
    }
    
    let config = unsafe { &mut *handle };
    let key = unsafe { CStr::from_ptr(key).to_str().ok() };
    let value = unsafe { CStr::from_ptr(value).to_str().ok() };
    
    match (key, value) {
        (Some(k), Some(v)) => {
            config.values.insert(k.to_string(), v.to_string());
            true
        }
        _ => false,
    }
}

#[no_mangle]
pub extern "C" fn config_free(handle: ConfigHandle) {
    if !handle.is_null() {
        unsafe { drop(Box::from_raw(handle)); }
    }
}
```


---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*