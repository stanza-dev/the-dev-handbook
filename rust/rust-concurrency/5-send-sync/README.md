# 🚚 Send & Sync Traits

> Part of [Concurrent Rust](https://stanza.dev/courses/rust-concurrency)

The foundation of Rust's compile-time thread safety. Learn how the Send and Sync marker traits prevent data races, understand which types are thread-safe and why, and master patterns like PhantomData, newtype wrappers, and the Arc<Mutex<T>> vs Arc<RwLock<T>> decision tree.

## Lessons

1. [The Send Trait](./1-send-trait.md)
2. [The Sync Trait](./2-sync-trait.md)
3. [Thread Safety Patterns](./3-safety-patterns.md)
4. [Interior Mutability & Thread Safety](./4-mutability-threads.md)

## Practice Challenges

This section includes **4 interactive challenges** available on Stanza:

- ✏️ Fill in the Blank: Arc<T> Trait Bound
- 🧩 Challenge: Making a Type Thread-Safe
- 🔗 Matching: Send & Sync Status
- 🧩 Multiple Choice: Mutex<T> and Sync

→ [Practice in your IDE](https://stanza.dev/courses/rust-concurrency)