---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-maybe-uninit"
---

# The Problem: Uninitialized Memory

```rust
// UB! Reading uninitialized memory
let x: i32;
println!("{x}");  // Undefined behavior!

// UB! Even creating "uninitialized" values
let x: bool = unsafe { std::mem::uninitialized() };  // DEPRECATED - UB!
// A bool must be 0 or 1; random bits are invalid
```

# MaybeUninit: Safe Uninitialized Memory

```rust
use std::mem::MaybeUninit;

// Safe: Explicitly uninitialized
let mut x: MaybeUninit<i32> = MaybeUninit::uninit();

// Write a value
x.write(42);

// Now safe to read
let value = unsafe { x.assume_init() };
println!("{value}");  // 42
```

## Initializing Arrays

```rust
use std::mem::MaybeUninit;

fn create_array() -> [String; 1000] {
    let mut arr: [MaybeUninit<String>; 1000] = 
        unsafe { MaybeUninit::uninit().assume_init() };
    
    for (i, elem) in arr.iter_mut().enumerate() {
        elem.write(format!("Item {i}"));
    }
    
    // Transmute to initialized array
    // SAFETY: All elements are initialized
    unsafe {
        std::mem::transmute::<_, [String; 1000]>(arr)
    }
}
```

## Common Patterns

```rust
// Reading into uninitialized buffer
fn read_exact(buf: &mut [MaybeUninit<u8>]) {
    // Fill buf with data...
}

// Out parameter pattern (like C APIs)
fn get_value(out: &mut MaybeUninit<BigStruct>) {
    out.write(BigStruct::new());
}
```

## Don't Use mem::zeroed() Carelessly

```rust
// DANGEROUS for many types!
let s: String = unsafe { std::mem::zeroed() };  // UB!
// String expects valid heap pointer, not null

let r: &i32 = unsafe { std::mem::zeroed() };  // UB!
// References can never be null
```

See [MaybeUninit](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html).

## Code Examples

**Generic array initialization**

```rust
use std::mem::MaybeUninit;

// Efficient array initialization
fn init_array<T, F, const N: usize>(mut f: F) -> [T; N]
where
    F: FnMut(usize) -> T,
{
    let mut arr: [MaybeUninit<T>; N] = unsafe {
        MaybeUninit::uninit().assume_init()
    };
    
    for (i, elem) in arr.iter_mut().enumerate() {
        elem.write(f(i));
    }
    
    // SAFETY: All elements initialized
    unsafe {
        // This is the recommended way in newer Rust:
        let ptr = arr.as_ptr() as *const [T; N];
        std::ptr::read(ptr)
    }
}

let squares: [i32; 10] = init_array(|i| (i * i) as i32);
assert_eq!(squares, [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]);

let strings: [String; 5] = init_array(|i| format!("str_{i}"));
assert_eq!(strings[2], "str_2");
```


## Resources

- [MaybeUninit](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html) — MaybeUninit documentation

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*