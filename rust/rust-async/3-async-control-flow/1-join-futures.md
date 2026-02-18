---
source_course: "rust-async"
source_lesson: "rust-async-join-futures"
---

# Running Futures Concurrently

## The Problem: Sequential is Slow

```rust
// Sequential - each waits for the previous
let a = fetch_a().await; // 100ms
let b = fetch_b().await; // 100ms
// Total: 200ms
```

## Solution: join!

```rust
let (a, b) = tokio::join!(fetch_a(), fetch_b());
// Both run concurrently, total: ~100ms
```

`join!` polls all futures concurrently and returns when **ALL** complete.

## try_join! for Results

When futures return `Result`, use `try_join!` for early error return:

```rust
use tokio::try_join;

async fn fetch_data() -> Result<(User, Posts), Error> {
    let (user, posts) = try_join!(fetch_user(), fetch_posts())?;
    // Returns early if either fails
    Ok((user, posts))
}
```

## JoinSet for Dynamic Tasks

```rust
use tokio::task::JoinSet;

let mut set = JoinSet::new();

for url in urls {
    set.spawn(async move {
        fetch(url).await
    });
}

// Process results as they complete
while let Some(result) = set.join_next().await {
    match result {
        Ok(data) => process(data),
        Err(e) => eprintln!("Task failed: {e}"),
    }
}
```

## futures::join_all for Homogeneous Futures

```rust
use futures::future::join_all;

let futures: Vec<_> = urls.iter().map(|url| fetch(url)).collect();
let results: Vec<Response> = join_all(futures).await;
```

## Code Examples

**Parallel API fetching**

```rust
// Real-world: parallel API fetching
async fn get_user_data(user_id: u64) -> Result<UserData, Error> {
    let (profile, posts, followers) = tokio::try_join!(
        fetch_profile(user_id),
        fetch_posts(user_id),
        fetch_followers(user_id),
    )?;
    
    Ok(UserData { profile, posts, followers })
}
```


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*