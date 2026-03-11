---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-lists"
---

# Lists: Ordered Collections

## Introduction

Redis Lists are linked lists of string values, supporting push/pop operations from both ends in O(1) time. They are the backbone of queues, stacks, activity feeds, and any pattern that requires ordered, sequential data.

## Key Concepts

- **Doubly linked list**: Redis Lists support O(1) operations at both head (LPUSH/LPOP) and tail (RPUSH/RPOP).
- **FIFO queue**: Push to one end (RPUSH) and pop from the other (LPOP) to implement first-in, first-out processing.
- **Blocking operations**: BLPOP and BRPOP block the client until data is available — ideal for worker queues.
- **Capped lists**: LTRIM keeps only the most recent N elements, useful for activity feeds.

## Real World Context

Lists power job queues (Sidekiq, Bull), recent activity feeds (last 100 actions), notification inboxes, and message buffers. BLPOP makes them efficient for worker patterns where consumers wait for new jobs without polling.

## Deep Dive

Redis Lists are linked lists of string values. They're perfect for implementing queues, stacks, and maintaining ordered data.

## List Basics

Lists maintain insertion order and allow duplicates:

```
┌───────────────────────────────────────────┐
│  HEAD                              TAIL   │
│   ↓                                  ↓    │
│  [A] ←→ [B] ←→ [C] ←→ [D] ←→ [E]         │
│   0      1      2      3      4           │
│  -5     -4     -3     -2     -1           │
└───────────────────────────────────────────┘
```

## Adding Elements

### LPUSH and RPUSH

```redis
# Push to left (head)
LPUSH tasks "task3"
LPUSH tasks "task2"
LPUSH tasks "task1"
# List is now: [task1, task2, task3]

# Push to right (tail)
RPUSH tasks "task4"
RPUSH tasks "task5"
# List is now: [task1, task2, task3, task4, task5]

# Push multiple at once
RPUSH mylist "a" "b" "c"
# Adds a, then b, then c to the right
```

## Reading Elements

### LRANGE

```redis
# Get range of elements (0-indexed)
LRANGE tasks 0 2      # First 3 elements
# Returns: ["task1", "task2", "task3"]

LRANGE tasks 0 -1     # All elements
# Returns: ["task1", "task2", "task3", "task4", "task5"]

LRANGE tasks -3 -1    # Last 3 elements
# Returns: ["task3", "task4", "task5"]
```

### LINDEX and LLEN

```redis
# Get element at index
LINDEX tasks 0        # Returns: "task1"
LINDEX tasks -1       # Returns: "task5" (last element)

# Get list length
LLEN tasks            # Returns: 5
```

## Removing Elements

### LPOP and RPOP

```redis
# Remove and return from left
LPOP tasks
# Returns: "task1", list is now [task2, task3, task4, task5]

# Remove and return from right
RPOP tasks
# Returns: "task5", list is now [task2, task3, task4]

# Pop multiple elements
LPOP tasks 2
# Returns: ["task2", "task3"]
```

### LREM - Remove by Value

```redis
RPUSH mylist "a" "b" "a" "c" "a"

# Remove 2 occurrences of "a" from head
LREM mylist 2 "a"     # Returns: 2

# Remove 1 occurrence from tail (negative count)
LREM mylist -1 "a"    # Returns: 1

# Remove all occurrences
LREM mylist 0 "a"     # Returns: number removed
```

## Implementing a Queue (FIFO)

```redis
# Producer: add jobs to queue
RPUSH job_queue "job:1"
RPUSH job_queue "job:2"
RPUSH job_queue "job:3"

# Consumer: process jobs in order
LPOP job_queue        # Returns: "job:1"
LPOP job_queue        # Returns: "job:2"
```

## Implementing a Stack (LIFO)

```redis
# Push items onto stack
LPUSH stack "first"
LPUSH stack "second"
LPUSH stack "third"

# Pop items (last in, first out)
LPOP stack            # Returns: "third"
LPOP stack            # Returns: "second"
```

## Blocking Operations

Block until data is available (great for job queues):

```redis
# Block for up to 30 seconds waiting for data
BLPOP job_queue 30
# Returns job when available, or nil after timeout

# Block on multiple lists
BLPOP queue1 queue2 queue3 30
```

## Trimming Lists

```redis
# Keep only elements 0-99 (last 100)
LTRIM recent_activity 0 99

# Useful pattern: add and trim in one go
LPUSH recent_activity "new_event"
LTRIM recent_activity 0 999    # Keep last 1000
```

📖 [List Commands Documentation](https://redis.io/docs/latest/commands/?group=list)

## Common Pitfalls

1. **Using LINDEX on large lists** — LINDEX is O(n) for elements far from the head. For random access patterns, use a hash or sorted set instead.
2. **Forgetting to LTRIM after LPUSH** — Without trimming, activity feed lists grow unboundedly. Always pair LPUSH with LTRIM for capped collections.

## Best Practices

1. **Use RPUSH + LPOP for FIFO queues** — This gives reliable first-in, first-out ordering.
2. **Use BLPOP for worker consumers** — Blocking pop eliminates polling and reduces latency for job processing.

## Summary

- Lists are doubly linked lists with O(1) push/pop at both ends.
- RPUSH + LPOP implements a FIFO queue; LPUSH + LPOP implements a stack.
- BLPOP blocks until data arrives — perfect for job queue workers.
- LTRIM caps list length to prevent unbounded growth.
- LRANGE retrieves slices of the list by index range.

## Code Examples

**List as a FIFO queue — RPUSH adds to the tail, LPOP removes from the head, LTRIM caps the list size**

```bash
# Build a job queue (FIFO)
RPUSH jobs "email:welcome" "email:verify" "email:report"

# Process jobs in order
LPOP jobs
# "email:welcome"

# Keep only last 100 items (capped list)
LPUSH recent_activity "new_event"
LTRIM recent_activity 0 99
```


## Resources

- [Redis Lists](https://redis.io/docs/latest/develop/data-types/lists/) — Complete guide to Redis List data type

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*