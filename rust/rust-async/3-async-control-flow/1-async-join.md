---
source_course: "rust-async"
source_lesson: "rust-async-join"
---

# Concurrent Execution with join!

## Introduction

By default, `.await` is sequential — each future runs one after another. `join!`, `try_join!`, `JoinSet`, and `join_all` let you run futures **concurrently** on the same task.

## Key Concepts

**Sequential (slow):**
```rust
let a = fetch_a().await; // 100ms
let b = fetch_b().await; // 100ms
// Total: ~200ms
```

**Concurrent with `join!`:**
```rust
let (a, b) = tokio::join!(fetch_a(), fetch_b());
// Total: ~100ms — both run concurrently
```

`join!` polls all futures on the **same task** (not separate threads) and returns when **all** complete.

## Real World Context

Parallel API calls, concurrent database queries, fetching a user's profile + posts + followers simultaneously — any time you need multiple independent I/O results.

## Deep Dive

**`try_join!`** for `Result`-returning futures — short-circuits on first error:

```rust
let (user, posts) = tokio::try_join!(
    fetch_user(id),
    fetch_posts(id),
)?; // Returns early if either fails
```

**`JoinSet`** for dynamic task counts:
```rust
let mut set = JoinSet::new();
for url in urls {
    set.spawn(async move { fetch(url).await });
}
while let Some(result) = set.join_next().await {
    handle(result?);
}
```

**`futures::future::join_all`** for homogeneous future collections:
```rust
let futures: Vec<_> = urls.iter().map(|u| fetch(u)).collect();
let results = futures::future::join_all(futures).await;
```

## Common Pitfalls

- Using `join!` when you need early error return — use `try_join!` instead
- Confusing `join!` (concurrent on one task) with `spawn` (separate tasks)
- `join_all` with thousands of futures — consider `buffer_unordered` instead

## Best Practices

- Use `join!` / `try_join!` for a fixed, small number of futures
- Use `JoinSet` for dynamic or large sets of spawned tasks
- Use `buffer_unordered` on streams for bounded concurrency

## Summary

`join!` runs futures concurrently and waits for all. `try_join!` adds early error return. `JoinSet` manages dynamic task groups. `join_all` handles homogeneous collections.

## Code Examples

**Fetching multiple resources concurrently with error handling**

```rust
// Parallel API fetching with try_join!
async fn get_dashboard(user_id: u64) -> Result<Dashboard, Error> {
    let (profile, posts, notifications) = tokio::try_join!(
        fetch_profile(user_id),
        fetch_posts(user_id),
        fetch_notifications(user_id),
    )?;

    Ok(Dashboard { profile, posts, notifications })
}
```


## Resources

- [Tokio join!](https://docs.rs/tokio/latest/tokio/macro.join.html) — Tokio join! macro documentation

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*