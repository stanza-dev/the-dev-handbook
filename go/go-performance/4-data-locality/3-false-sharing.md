---
source_course: "go-performance"
source_lesson: "go-performance-false-sharing"
---

# False Sharing

When two goroutines access different variables on the same cache line, they can slow each other down (cache line bouncing).

## The Problem

```go
type Counters struct {
    A int64  // Goroutine 1 writes
    B int64  // Goroutine 2 writes (same cache line!)
}
```

## The Solution: Padding

```go
type Counters struct {
    A int64
    _ [56]byte  // Pad to separate cache line
    B int64
}
```

## Using atomic Types

Go 1.19+ atomic types are already padded:

```go
type Counters struct {
    A atomic.Int64
    B atomic.Int64
}
```

## Detection

False sharing shows up as high cache miss rates in profiles. Look for unexpectedly slow atomic operations.

## Code Examples

**Padding for Cache Lines**

```go
// Padded to avoid false sharing
type PaddedCounter struct {
    value int64
    _     [56]byte  // Fill to 64-byte cache line
}

type Counters struct {
    counters [8]PaddedCounter  // Each on own cache line
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*