---
source_course: "rust-traits"
source_lesson: "rust-traits-marker-traits"
---

# Marker Traits

## Introduction
Marker traits have no methods — they mark types with compile-time properties. Rust's auto traits (`Send`, `Sync`, `Copy`, `Sized`, `Unpin`) are marker traits that the compiler implements automatically. Since Rust 1.85, the new `AsyncFn`, `AsyncFnMut`, and `AsyncFnOnce` traits join the prelude for async closure support.

## Key Concepts
- **Marker Trait**: A trait with no methods that marks a type with a property.
- **Auto Trait**: A marker trait automatically implemented by the compiler if all fields satisfy it.
- **Negative Impl**: Explicitly opting out of an auto trait: `impl !Send for MyType {}`.
- **Sealed Trait**: A pattern that prevents external crates from implementing your trait.

## Real World Context
`Send` and `Sync` are the foundation of Rust's thread safety. The compiler checks these markers to prevent data races at compile time. `Copy` marks types that can be duplicated by simple bit copying. Custom marker traits are used to create type-level permissions and capabilities.

## Deep Dive

### Compiler-Provided Markers

```rust
trait Send {}    // Safe to transfer between threads
trait Sync {}    // Safe to share references between threads
trait Copy {}    // Bitwise copyable (`Copy` itself has no methods, but it requires `Clone` which does)
trait Sized {}   // Known size at compile time
trait Unpin {}   // Safe to move after pinning
```

`Sized` has special compiler treatment: it is implicitly assumed for all type parameters unless opted out with `?Sized`. This is unlike other auto traits which are simply inferred from field types.

Auto traits are implemented automatically if all fields satisfy them:

```rust
struct MySendType {
    x: i32,      // Send ✓
    s: String,   // Send ✓
} // MySendType: Send ✓ (auto-implemented)
```

### Custom Marker Traits

```rust
trait JsonSerializable {}

impl JsonSerializable for i32 {}
impl JsonSerializable for String {}
impl<T: JsonSerializable> JsonSerializable for Vec<T> {}

fn to_json<T: JsonSerializable>(value: &T) -> String {
    todo!() // Can only be called with marked types
}
```

### Sealed Traits

Prevent external crates from implementing your trait:

```rust
mod private {
    pub trait Sealed {}
}

pub trait MyTrait: private::Sealed {
    fn method(&self);
}

// Only types you mark as Sealed can implement MyTrait
impl private::Sealed for MyType {}
impl MyTrait for MyType {
    fn method(&self) { /* ... */ }
}
```

### AsyncFn Traits (Rust 1.85+)

While not marker traits themselves (they have methods), several important traits were added to the Rust 1.85 prelude that are worth mentioning alongside marker traits.

Rust 1.85 added `AsyncFn`, `AsyncFnMut`, and `AsyncFnOnce` to the prelude. These are the async counterparts of `Fn`, `FnMut`, and `FnOnce`:

```rust
async fn call_async(f: impl AsyncFn(i32) -> String) -> String {
    f(42).await
}
```

Note that `AsyncFn*` traits are NOT dyn-compatible.

## Common Pitfalls
1. **Assuming all traits have methods** — Marker traits intentionally have no methods. Their value is in the type-level contract they express.
2. **Negative impls in stable Rust** — `impl !Send for T {}` is only available on nightly. On stable, use `PhantomData<*const ()>` to opt out of both `Send` and `Sync` (raw pointers are neither).

## Best Practices
1. **Use sealed traits for public API stability** — Sealing prevents external breakage when you add methods to the trait.
2. **Prefer auto-derived markers** — Let the compiler derive `Send` and `Sync` automatically; only add manual impls when wrapping raw pointers or FFI types.

## Summary
- Marker traits have no methods and mark types with compile-time properties.
- Auto traits (`Send`, `Sync`, `Copy`, `Sized`, `Unpin`) are implemented automatically by the compiler.
- Sealed traits prevent external implementations using a private supertrait.
- `AsyncFn`, `AsyncFnMut`, `AsyncFnOnce` (Rust 1.85) are the async closure traits.

## Code Examples

**Custom marker traits combined with PhantomData to create compile-time permission checks — a read-only file type cannot call write methods**

```rust
// Custom marker trait for type-level permissions
trait CanRead {}
trait CanWrite {}

struct File<R, W> {
    path: String,
    _read: std::marker::PhantomData<R>,
    _write: std::marker::PhantomData<W>,
}

struct Yes;
struct No;
impl CanRead for Yes {}
impl CanWrite for Yes {}

impl<W> File<Yes, W> {
    fn read(&self) -> Vec<u8> { todo!() } // Only if R = Yes
}

impl<R> File<R, Yes> {
    fn write(&self, _data: &[u8]) { todo!() } // Only if W = Yes
}

// Compile-time enforced: read-only file cannot write
let ro_file: File<Yes, No> = todo!();
ro_file.read(); // ✓
// ro_file.write(b"data"); // ✗ Compile error!
```


## Resources

- [Marker Traits](https://doc.rust-lang.org/std/marker/index.html) — Rust standard library documentation for the marker module

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*