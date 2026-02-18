---
source_course: "go-std-lib"
source_lesson: "go-std-lib-time-duration"
---

# The Time Package

`time.Time` represents an instant in time. `time.Duration` represents an elapsed time.

## Creating Times

```go
now := time.Now()
specific := time.Date(2024, time.January, 1, 12, 0, 0, 0, time.UTC)
```

## Durations

```go
d := 5 * time.Second
d := 100 * time.Millisecond
d := 2 * time.Hour + 30 * time.Minute
```

## Monotonic Time
Go's `time.Now()` contains a monotonic clock reading for accurate duration measurements, unaffected by system clock changes (e.g. NTP updates).

```go
start := time.Now()
// do work
elapsed := time.Since(start)  // Uses monotonic clock
```

## Time Arithmetic

```go
future := now.Add(24 * time.Hour)
past := now.Add(-1 * time.Hour)
diff := t1.Sub(t2)  // Returns Duration
```

## Comparison

```go
t1.Before(t2)
t1.After(t2)
t1.Equal(t2)
```

## Code Examples

**Duration Usage**

```go
d := 10 * time.Second
fmt.Println(d.Milliseconds()) // 10000
fmt.Println(d.String())       // 10s
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*