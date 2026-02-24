---
source_course: "go-std-lib"
source_lesson: "go-std-lib-time-duration"
---

# Time and Duration

## Introduction
Every real-world program eventually needs to deal with time: scheduling tasks, measuring latency, or enforcing deadlines. Go's `time` package gives you two core building blocks, `time.Time` for instants and `time.Duration` for elapsed spans, and understanding both is essential for writing correct, production-grade code.

## Key Concepts
- **time.Time**: An immutable value representing a single instant, carrying both a wall-clock reading and a monotonic clock reading.
- **time.Duration**: A signed, nanosecond-precision span of time stored internally as an `int64`.
- **Monotonic clock**: A clock that only moves forward, unaffected by NTP adjustments or manual system clock changes.

## Real World Context
Imagine you are building an HTTP server and you want to log how long each request takes. If the system clock jumps backward due to an NTP correction between the start and end of a request, a naive wall-clock subtraction would give a negative or wildly wrong duration. Go silently includes a monotonic reading in `time.Now()` so that `time.Since(start)` always returns an accurate elapsed time, even across clock adjustments.

## Deep Dive
You create a `time.Time` with `time.Now()` for the current moment or `time.Date()` for a specific point in time.

```go
now := time.Now()
specific := time.Date(2024, time.January, 1, 12, 0, 0, 0, time.UTC)
```

Both calls return a `time.Time`. The `time.Date` constructor lets you pin down year, month, day, hour, minute, second, nanosecond, and location.

Durations are built by multiplying typed constants. Because `time.Duration` is nanosecond-based, you never work with raw integers. Instead you compose human-readable expressions.

```go
d := 5 * time.Second
d := 100 * time.Millisecond
d := 2 * time.Hour + 30 * time.Minute
```

These expressions are type-safe. You cannot accidentally add a plain `int` to a `time.Duration` without an explicit conversion.

Go's `time.Now()` embeds a monotonic clock reading alongside the wall clock. Functions like `time.Since` and the `Sub` method automatically use the monotonic component when both operands carry one.

```go
start := time.Now()
// do work
elapsed := time.Since(start) // Uses monotonic clock
```

This means your measurements are immune to NTP updates, leap-second adjustments, or any wall-clock skew.

Time arithmetic lets you shift instants forward or backward and compute the gap between two instants.

```go
future := now.Add(24 * time.Hour)
past := now.Add(-1 * time.Hour)
diff := t1.Sub(t2) // Returns Duration
```

The `Add` method shifts a `time.Time`, while `Sub` returns a `time.Duration`. These are the only arithmetic operations you need.

Comparing two `time.Time` values is straightforward.

```go
t1.Before(t2)
t1.After(t2)
t1.Equal(t2)
```

Always prefer `Equal` over `==` because `Equal` correctly handles different time zone representations of the same instant.

## Common Pitfalls
1. **Using `==` instead of `Equal`** — Two `time.Time` values can represent the same instant but carry different internal locations. The `==` operator compares locations too, so it may return `false` for logically identical times.
2. **Multiplying durations by int variables** — Writing `time.Second * n` where `n` is an `int` variable will not compile. You need `time.Duration(n) * time.Second`.
3. **Ignoring monotonic stripping on serialization** — The monotonic reading is stripped when you marshal a `time.Time` (e.g., to JSON). If you unmarshal and then call `Sub`, you get a wall-clock difference, not a monotonic one.

## Best Practices
1. **Use `time.Since(start)` for elapsed time** — It reads the monotonic clock and is the idiomatic one-liner for benchmarking or logging durations.
2. **Prefer typed duration constants over raw numbers** — Write `5 * time.Second` instead of `time.Duration(5000000000)`. The constants are self-documenting and prevent magnitude errors.

## Summary
- `time.Time` represents an instant; `time.Duration` represents a span.
- Go transparently records a monotonic clock reading in `time.Now()` to protect elapsed-time measurements.
- Use `Add`/`Sub` for arithmetic and `Equal` (not `==`) for comparison.
- Build durations from typed constants like `time.Second` and `time.Millisecond` for clarity and safety.

## Code Examples

**Creating and using time.Duration values — multiplication with typed constants and parsing duration strings from configuration**

```go
d := 10 * time.Second
fmt.Println(d.Milliseconds()) // 10000
fmt.Println(d.String())       // 10s
```


## Resources

- [time package - Go Documentation](https://pkg.go.dev/time) — Official Go documentation for time, duration, and timer operations

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*