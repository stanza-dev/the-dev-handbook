---
source_course: "redis-scripting"
source_lesson: "redis-scripting-watch-deep"
---

# Check-and-Set with WATCH

## Introduction

WATCH enables the check-and-set (CAS) pattern in Redis — read a value, compute a new value, and write it back only if the original value has not changed. This is the foundation for correct concurrent updates without pessimistic locking.

## Key Concepts

- **Check-and-Set (CAS)** — read → compute → conditionally write; abort if someone else wrote in between
- **Optimistic locking** — assumes most transactions will not conflict; retries on the rare conflict
- **WATCH scope** — watches apply per-connection; different clients each manage their own watches
- **Retry loop** — the application must loop until EXEC succeeds (returns non-nil)

## Real World Context

A leaderboard service needs to update a player's high score only if the new score beats the current one. Without CAS, two concurrent updates for the same player could both read the current score, both compute a new score, and both write — the lower update could overwrite the higher one.

## Deep Dive

The canonical WATCH retry loop in Python:

```python
import redis

r = redis.Redis()

def update_high_score(player_id, new_score):
    key = f"score:{player_id}"
    while True:
        with r.pipeline() as pipe:
            try:
                pipe.watch(key)
                current = pipe.get(key)
                current_score = int(current) if current else 0
                if new_score <= current_score:
                    pipe.unwatch()
                    return False  # No update needed
                pipe.multi()
                pipe.set(key, new_score)
                pipe.execute()  # Returns None if watched key changed
                return True
            except redis.WatchError:
                continue  # Retry
```

The redis-py client raises `WatchError` when EXEC returns nil, making the retry loop clean to implement.

Note: `redis.WatchError` is a redis-py (Python client) convenience exception. At the Redis protocol level, a failed WATCH causes EXEC to return nil — client libraries may wrap this as an exception, a null check, or a special return value depending on the language.

A raw Redis CLI session showing a conflict scenario:

```bash
# Client A
WATCH score:player1
GET score:player1
# => 100

# Client B (interleaves here)
SET score:player1 200

# Client A resumes
MULTI
SET score:player1 150
EXEC
# => (nil)  -- Client A's transaction aborted
```

## Common Pitfalls

1. **Unbounded retry loops.** Under extreme contention, a loop with no limit could spin forever. Always add a maximum retry count and back off.
2. **Not calling UNWATCH before returning early.** If you decide not to proceed with the transaction, call UNWATCH to release the watch on the current connection.
3. **Watching volatile keys.** If a watched key expires between WATCH and EXEC, Redis treats the expiry as a modification and aborts the transaction.

## Best Practices

1. Cap retry loops at 3-5 attempts; after that, return a conflict error to the caller.
2. Under sustained high contention (>50% retry rate), switch to Lua scripting — it handles the check-and-set atomically without retries.
3. Watch only the minimal set of keys required for correctness; each watched key adds abort risk.

## Summary

- WATCH implements check-and-set: read before MULTI, write inside MULTI/EXEC
- EXEC returning nil means a watched key changed; the application must retry
- redis-py raises WatchError on nil EXEC, simplifying retry loop code
- Always unwatch explicitly when aborting before MULTI
- High-contention workloads should consider Lua scripting as an alternative

## Code Examples

**WATCH retry loop in Python (redis-py)**

```python
import redis

r = redis.Redis()

def update_high_score(player_id, new_score):
    key = f"score:{player_id}"
    while True:
        with r.pipeline() as pipe:
            try:
                pipe.watch(key)
                current = pipe.get(key)
                current_score = int(current) if current else 0
                if new_score <= current_score:
                    pipe.unwatch()
                    return False
                pipe.multi()
                pipe.set(key, new_score)
                pipe.execute()
                return True
            except redis.WatchError:
                continue
```


## Resources

- [WATCH Command](https://redis.io/commands/watch/) — WATCH command reference

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*