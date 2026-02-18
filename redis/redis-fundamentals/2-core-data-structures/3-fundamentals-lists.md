---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-lists"
---

# Lists: Ordered Collections

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

## Resources

- [Redis Lists](https://redis.io/docs/latest/develop/data-types/lists/) — Complete guide to Redis List data type

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*