---
source_course: "rust-traits"
source_lesson: "rust-traits-dyn-compat-rules"
---

# Dyn Compatibility in Depth

## Introduction
Understanding exactly which traits can be used as trait objects is critical for designing flexible APIs. This lesson explores the dyn compatibility rules (formerly called "object safety") in detail, including workarounds for common violations.

## Key Concepts
- **Dyn Compatibility**: A trait is dyn-compatible when every method can be dispatched through a vtable without knowing the concrete type.
- **Self: Sized Escape Hatch**: Adding `where Self: Sized` to a method excludes it from the vtable, keeping the trait dyn-compatible.
- **Dispatchable Methods**: Methods that can be called through a trait object because they do not depend on the concrete type's identity.

## Real World Context
Library authors frequently need to make traits dyn-compatible so users can store heterogeneous collections. The `Error` trait, `Iterator` (via `dyn Iterator<Item = T>`), and GUI widget traits all need careful design to remain dyn-compatible.

## Deep Dive

### The Six Rules

A trait is dyn-compatible when:

1. **No `Self: Sized` supertrait** — The trait itself must not require `Self: Sized`.
2. **No associated constants** — Constants cannot be dispatched through a vtable.
3. **Every method has a receiver** — `&self`, `&mut self`, `self`, `Box<Self>`, `Rc<Self>`, `Arc<Self>`, or `Pin<&mut Self>`.
4. **No method returns `Self`** — The vtable cannot know the concrete return type's size.
5. **No generic methods** — Type parameters would require infinite vtable entries.
6. **No `-> impl Trait` or `async fn`** — These hide the concrete return type, which cannot be dispatched dynamically.

### The `where Self: Sized` Workaround

You can keep a trait dyn-compatible by excluding problematic methods:

```rust
trait Cloneable {
    fn clone_box(&self) -> Box<dyn Cloneable>;

    // This method cannot be dispatched dynamically,
    // but the trait remains dyn-compatible
    fn into_owned(self) -> Self
    where
        Self: Sized;
}
```

Methods with `where Self: Sized` are not available on `dyn Cloneable`, but all other methods are.

### Making Non-Compatible Traits Usable

Wrap a non-compatible trait in a compatible one:

```rust
// Not dyn-compatible: returns Self
trait Duplicatable {
    fn duplicate(&self) -> Self;
}

// Dyn-compatible wrapper
trait DynDuplicatable {
    fn duplicate_boxed(&self) -> Box<dyn DynDuplicatable>;
}

impl<T: Duplicatable + Clone + 'static> DynDuplicatable for T {
    fn duplicate_boxed(&self) -> Box<dyn DynDuplicatable> {
        Box::new(self.clone())
    }
}
```

### Static Methods and Constructors

Static methods (no `self` receiver) break dyn compatibility. Move them behind `where Self: Sized`:

```rust
trait Factory {
    fn create() -> Self where Self: Sized;
    fn name(&self) -> &str; // This is fine for dyn dispatch
}
```

## Common Pitfalls
1. **Adding a generic method later** — Adding `fn process<T>(&self, val: T)` to a public trait is a breaking change if users depend on `dyn Trait`.
2. **Forgetting about associated constants** — Even a `const VERSION: u32 = 1;` in a trait breaks dyn compatibility.

## Best Practices
1. **Design traits for dyn compatibility from the start** — If you expect users to create trait objects, follow the rules from day one.
2. **Use `where Self: Sized` liberally** — It lets you add convenience methods without sacrificing dyn compatibility.
3. **Test with `fn assert_dyn_compat(_: &dyn YourTrait) {}`** — A compile-time check that your trait remains dyn-compatible.

## Summary
- Six rules determine dyn compatibility: no Sized supertrait, no associated constants, receiver on all methods, no Self returns, no generics, no impl Trait returns.
- `where Self: Sized` excludes individual methods from the vtable.
- Wrapper traits can make non-compatible traits usable as trait objects.
- Design public traits for dyn compatibility proactively.

## Code Examples

**The where Self: Sized escape hatch lets you add non-dispatchable methods to a trait without breaking dyn compatibility**

```rust
// Keeping a trait dyn-compatible with where Self: Sized
trait Plugin {
    fn name(&self) -> &str;
    fn execute(&self);

    // Excluded from dyn dispatch, but trait stays compatible
    fn clone_plugin(&self) -> Self
    where
        Self: Sized;
}

// This compiles: Plugin is dyn-compatible
fn run_plugins(plugins: &[Box<dyn Plugin>]) {
    for p in plugins {
        println!("Running: {}", p.name());
        p.execute();
        // p.clone_plugin(); // Not available on dyn Plugin
    }
}
```


## Resources

- [Dyn Compatibility](https://doc.rust-lang.org/reference/items/traits.html#object-safety) — Rust Reference on the rules for dyn-compatible traits

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*