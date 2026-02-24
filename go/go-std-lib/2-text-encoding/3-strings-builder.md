---
source_course: "go-std-lib"
source_lesson: "go-std-lib-strings-builder"
---

# Strings & Bytes

## Introduction

Strings in Go are immutable sequences of bytes. Every time you concatenate with `+=`, Go allocates a new string and copies all the data, leading to O(N²) performance in loops. The `strings` and `bytes` packages provide efficient alternatives: `strings.Builder` for building strings incrementally, and a rich set of utility functions for searching, splitting, and transforming text. If you write any Go code that processes text, you will use these packages daily.

## Key Concepts

- **Immutable strings**: Go strings cannot be modified in place. Any "modification" creates a new string.
- **strings.Builder**: A write-only buffer optimized for building strings incrementally with amortized O(N) performance.
- **bytes.Buffer**: An in-memory buffer for byte slices that implements both `io.Reader` and `io.Writer`.
- **strings package functions**: A collection of utilities for searching, splitting, joining, and transforming strings.

## Real World Context

You are building a template engine that constructs HTML emails. Each email contains a header, a dynamic body with user data, and a footer. Concatenating these parts with `+=` in a loop over thousands of users would be painfully slow. Using `strings.Builder`, you write each part once into a growable buffer and call `String()` at the end—the entire email is built in a single pass with minimal allocations.

## Deep Dive

### Why String Concatenation Is Slow

Because strings are immutable, every `+=` operation allocates a new string that is the combined length of the old string plus the new part, then copies both into the new allocation.

```go
s := ""
for i := 0; i < 10000; i++ {
    s += "a" // Copies the entire string every iteration!
}
```

On iteration 1, this copies 1 byte. On iteration 2, it copies 2 bytes. On iteration N, it copies N bytes. The total work is 1+2+3+...+N = O(N²). For 10,000 iterations, that is roughly 50 million byte copies.

### strings.Builder

`strings.Builder` solves this by maintaining an internal byte slice that grows as needed, similar to `append` for slices.

```go
var sb strings.Builder
for i := 0; i < 10000; i++ {
    sb.WriteString("a")
}
result := sb.String()
```

The Builder doubles its capacity when full, so the total work is O(N) with amortized constant-time appends. For 10,000 iterations, this is roughly 10,000 byte copies—a 5,000x improvement.

### Builder Methods

The Builder provides several write methods for different data types.

```go
var sb strings.Builder
sb.WriteString("Hello")    // Append a string
sb.WriteByte(',')          // Append a single byte
sb.WriteRune(' ')          // Append a Unicode code point
sb.WriteString("World!")
result := sb.String()      // Get the final string
sb.Reset()                 // Clear and reuse the builder
```

Each method returns the number of bytes written and an error (which is always nil for Builder, but satisfies the `io.Writer` interface).

### strings Package Utilities

The `strings` package provides a comprehensive set of functions for working with strings.

```go
strings.Contains(s, "needle")      // Check if substring exists
strings.HasPrefix(s, "Go")         // Check prefix
strings.HasSuffix(s, ".go")        // Check suffix
strings.Split(s, ",")              // Split into slice
strings.Join(slice, ", ")          // Join slice into string
strings.ToLower(s)                 // Convert to lowercase
strings.ToUpper(s)                 // Convert to uppercase
strings.TrimSpace(s)               // Remove leading/trailing whitespace
strings.ReplaceAll(s, "old", "new") // Replace all occurrences
strings.Count(s, "a")              // Count non-overlapping occurrences
strings.Index(s, "sub")            // Find first occurrence (-1 if not found)
```

These functions are pure—they return new strings without modifying the input.

### bytes.Buffer vs strings.Builder

Both accumulate data, but they serve different purposes.

```go
// bytes.Buffer: implements io.Reader AND io.Writer
var buf bytes.Buffer
buf.WriteString("data")
data, _ := io.ReadAll(&buf) // Can read back

// strings.Builder: write-only, optimized for string output
var sb strings.Builder
sb.WriteString("data")
s := sb.String() // Can only get the final string
```

Use `bytes.Buffer` when you need to both write and read (e.g., capturing output in tests). Use `strings.Builder` when you only need to build a string—it is slightly faster because it avoids the overhead of tracking read position.

### strings.NewReplacer

For multiple replacements, `strings.NewReplacer` is more efficient than chaining `strings.ReplaceAll` calls because it makes a single pass through the string.

```go
r := strings.NewReplacer(
    "&", "&amp;",
    "<", "&lt;",
    ">", "&gt;",
)
safe := r.Replace(userInput)
```

This is commonly used for HTML escaping and template variable substitution.

## Common Pitfalls

1. **Using `+=` in a loop** — This is the classic O(N²) mistake. Always use `strings.Builder` or `strings.Join` for concatenation in loops.
2. **Calling `sb.String()` in a loop** — Each call to `String()` on a Builder creates a copy of the internal buffer. If you need intermediate results, consider using `bytes.Buffer` instead.
3. **Confusing `strings.Split` edge cases** — `strings.Split("", ",")` returns `[""]` (a slice with one empty string), not an empty slice. Always check for empty input before splitting.

## Best Practices

1. **Use `strings.Join` for simple concatenation of slices** — It pre-calculates the total length and allocates once, making it the fastest way to join a string slice.
2. **Pre-grow the Builder with `sb.Grow(n)` when you know the final size** — This avoids repeated reallocations. For example, if you are building a 1KB string, call `sb.Grow(1024)` before writing.
3. **Use `strings.EqualFold` for case-insensitive comparison** — It is faster than converting both strings to lowercase and comparing.

## Summary

- Strings in Go are immutable; concatenation with `+=` in a loop is O(N²).
- `strings.Builder` provides O(N) string building with `WriteString`, `WriteByte`, and `WriteRune`.
- The `strings` package offers `Contains`, `Split`, `Join`, `TrimSpace`, `ReplaceAll`, and many other utilities.
- Use `bytes.Buffer` when you need both Reader and Writer; use `strings.Builder` when you only need to produce a string.
- Pre-allocate with `sb.Grow(n)` and use `strings.Join` for known-size concatenation to minimize allocations.

## Code Examples

**Building a string efficiently with strings.Builder — each WriteString appends to an internal buffer, and String() returns the final result without extra allocations**

```go
import "strings"

func join(strs []string) string {
    var sb strings.Builder
    for _, s := range strs {
        sb.WriteString(s)
    }
    return sb.String()
}
```


## Resources

- [strings package - Go Documentation](https://pkg.go.dev/strings) — Official reference for strings.Builder, strings.NewReader, and string utilities
- [bytes package - Go Documentation](https://pkg.go.dev/bytes) — Official reference for bytes.Buffer and byte slice operations

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*