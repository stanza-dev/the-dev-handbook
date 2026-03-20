---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-ffi-error-handling"
---

# FFI Error Handling & Panics

## Introduction
C functions communicate errors through return codes, errno, and sentinel values. Rust uses `Result` and `Option`. Bridging these two error models correctly is essential for robust FFI code. Additionally, Rust panics must never cross the FFI boundary — doing so is undefined behavior.

## Key Concepts
- **errno**: A thread-local error code set by many C functions on failure.
- **Sentinel value**: A special return value (like `-1` or `NULL`) that indicates an error.
- **Unwinding across FFI**: A Rust panic that propagates through a C stack frame is undefined behavior.
- **`catch_unwind`**: A function that catches panics, preventing them from crossing the FFI boundary.

## Real World Context
Every production FFI binding must handle errors. The `nix` crate wraps POSIX errno into Rust errors. Libraries like `rusqlite` convert SQLite error codes into `Result`. Forgetting to handle panics in callbacks leads to crashes that are impossible to debug.

## Deep Dive

### Converting C Error Codes to Result

Many C functions return 0 for success and -1 for failure, setting `errno`:

```rust
use std::io;

unsafe extern "C" {
    fn chdir(path: *const libc::c_char) -> libc::c_int;
}

fn change_directory(path: &str) -> io::Result<()> {
    let c_path = std::ffi::CString::new(path)
        .map_err(|_| io::Error::new(io::ErrorKind::InvalidInput, "null byte"))?;

    // SAFETY: c_path is a valid null-terminated string
    let result = unsafe { chdir(c_path.as_ptr()) };

    if result == 0 {
        Ok(())
    } else {
        Err(io::Error::last_os_error())
    }
}
```

`io::Error::last_os_error()` reads errno and converts it to a Rust error. This must be called immediately after the C function, before any other calls that might change errno.

### Preventing Panics from Crossing FFI

Rust panics that unwind through C frames are undefined behavior. Use `catch_unwind` in Rust functions called from C:

```rust
use std::panic;

#[unsafe(no_mangle)]
pub extern "C" fn callback(data: *const u8) -> i32 {
    let result = panic::catch_unwind(|| {
        // This code might panic
        process_data(data)
    });

    match result {
        Ok(value) => value,
        Err(_) => -1, // Return error code instead of unwinding
    }
}
```

The `catch_unwind` boundary ensures the panic is caught and converted to an error code before it reaches C.

### Handling Nullable Return Values

Wrap nullable C return values in `Option`:

```rust
unsafe extern "C" {
    fn dlopen(filename: *const libc::c_char, flags: libc::c_int) -> *mut libc::c_void;
    fn dlerror() -> *const libc::c_char;
}

fn load_library(path: &str) -> Result<*mut libc::c_void, String> {
    let c_path = std::ffi::CString::new(path).map_err(|e| e.to_string())?;

    // SAFETY: c_path is a valid null-terminated string
    let handle = unsafe { dlopen(c_path.as_ptr(), libc::RTLD_NOW) };

    if handle.is_null() {
        // SAFETY: dlerror returns a valid string or null
        let err = unsafe {
            let msg = dlerror();
            if msg.is_null() {
                "unknown error".to_string()
            } else {
                std::ffi::CStr::from_ptr(msg).to_string_lossy().into_owned()
            }
        };
        Err(err)
    } else {
        Ok(handle)
    }
}
```

This pattern — check for null, then read the error message — is standard for C APIs.

## Common Pitfalls
1. **Reading errno too late** — Any intervening function call (including `println!`) can overwrite errno. Read it immediately after the C call.
2. **Letting panics cross FFI** — This is instant undefined behavior. Always use `catch_unwind` in Rust functions called from C.
3. **Ignoring return codes** — Many C functions silently return error codes. Always check the return value.

## Best Practices
1. **Create an error enum for the C library** — Map all possible error codes to a Rust enum that implements `std::error::Error`.
2. **Always use `catch_unwind` in `extern "C"` functions** — This is especially critical for callbacks passed to C code.
3. **Read errno immediately** — Store it in a variable before doing anything else.

## Summary
- Convert C error codes (return values, errno) to `Result` types.
- Read errno immediately after the C call using `io::Error::last_os_error()`.
- Use `catch_unwind` to prevent panics from crossing the FFI boundary.
- Check all return values from C functions — null pointers and negative integers often indicate errors.
- Create dedicated error types for clean error propagation.

## Code Examples

**Safe wrapper for POSIX open/read/close with proper error handling — converts C error codes and errno to io::Result**

```rust
use std::ffi::CString;
use std::io;

unsafe extern "C" {
    fn open(path: *const libc::c_char, flags: libc::c_int) -> libc::c_int;
    fn close(fd: libc::c_int) -> libc::c_int;
    fn read(fd: libc::c_int, buf: *mut libc::c_void, count: libc::size_t) -> libc::ssize_t;
}

/// Read up to `buf.len()` bytes from a file. Returns bytes read.
fn read_file(path: &str, buf: &mut [u8]) -> io::Result<usize> {
    let c_path = CString::new(path)
        .map_err(|_| io::Error::new(io::ErrorKind::InvalidInput, "null byte in path"))?;

    // SAFETY: c_path is valid, O_RDONLY is a valid flag
    let fd = unsafe { open(c_path.as_ptr(), libc::O_RDONLY) };
    if fd < 0 {
        return Err(io::Error::last_os_error()); // Read errno immediately!
    }

    // SAFETY: fd is valid, buf points to buf.len() writable bytes
    let bytes_read = unsafe {
        read(fd, buf.as_mut_ptr() as *mut libc::c_void, buf.len())
    };

    // SAFETY: fd is a valid open file descriptor
    unsafe { close(fd); }

    if bytes_read < 0 {
        Err(io::Error::last_os_error())
    } else {
        Ok(bytes_read as usize)
    }
}
```


## Resources

- [std::panic::catch_unwind](https://doc.rust-lang.org/std/panic/fn.catch_unwind.html) — API reference for catching panics before they cross FFI boundaries

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*