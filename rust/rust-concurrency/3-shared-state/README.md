# 🔒 Shared State Concurrency

> Part of [Concurrent Rust](https://stanza.dev/courses/rust-concurrency)

Mutexes, RwLocks, and synchronization primitives for shared mutable state. Master the Arc<Mutex<T>> pattern, learn when RwLock outperforms Mutex, use condition variables for signaling, and leverage one-time initialization with OnceLock and LazyLock.

## Lessons

1. [Mutex<T>: Mutual Exclusion](./1-mutex-basics.md)
2. [RwLock & Condvar](./2-rwlock-condvar.md)
3. [Once, LazyLock & Barrier](./3-lazy-barrier.md)
4. [Poisoned Mutexes & Recovery](./4-poisoned-mutexes.md)

## Practice Challenges

This section includes **4 interactive challenges** available on Stanza:

- ✏️ Fill in the Blank: Mutex Guard Release
- 🧩 Multiple Choice: Mutex Poisoning
- 🧩 Challenge: Condvar Wait Pattern
- 🧩 Challenge: Sync Primitives True Statements

→ [Practice in your IDE](https://stanza.dev/courses/rust-concurrency)