---
source_course: "go-concurrency"
source_lesson: "go-concurrency-closure-gotchas"
---

# Loop Variable Capture

A classic Go gotcha: closures in goroutines capture variables by reference.

## The Problem (Pre-Go 1.22)

```go
for i := 0; i < 5; i++ {
    go func() {
        fmt.Println(i)  // All might print 5!
    }()
}
```

## Solutions

### Pass as Argument

```go
for i := 0; i < 5; i++ {
    go func(n int) {
        fmt.Println(n)  // Correct: 0, 1, 2, 3, 4
    }(i)
}
```

### Shadow Variable

```go
for i := 0; i < 5; i++ {
    i := i  // Shadow with new variable
    go func() {
        fmt.Println(i)
    }()
}
```

## Go 1.22+ Fix

Go 1.22 changed loop variable semantics. Each iteration creates a new variable, fixing this issue automatically. But understanding the problem is still important for legacy code.

## Code Examples

**Safe Loop Variable Capture**

```go
// Correct pattern
for _, url := range urls {
    url := url  // Important: create new variable
    go func() {
        fetch(url)
    }()
}
```


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*