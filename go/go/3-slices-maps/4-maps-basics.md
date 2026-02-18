---
source_course: "go"
source_lesson: "go-maps-basics"
---

# Maps

## Introduction

Maps are Go's built-in hash table implementation, providing fast key-value storage and lookup. They're essential for caches, indexes, configurations, and any scenario requiring associative access. Understanding map semantics—especially nil maps and concurrent access—prevents common runtime panics.

## Key Concepts

**Hash Table**: Data structure providing O(1) average-case lookup, insertion, and deletion.

**Reference Type**: Maps are pointers to underlying data; passing to functions shares the map.

**Comma-Ok Idiom**: Pattern `v, ok := m[key]` distinguishing missing keys from zero values.

**Not Concurrency-Safe**: Maps require external synchronization for concurrent read/write access.

## Real World Context

Maps power caches, session stores, configuration lookup, counting/grouping, and graph adjacency lists. The random iteration order prevents code from accidentally depending on insertion order (a common source of test flakiness). The nil map panic catches uninitialized map bugs at development time.

## Deep Dive

### Creating Maps

```go
// Using make
m := make(map[string]int)

// With initial capacity (optimization)
m := make(map[string]int, 100)

// Map literal
m := map[string]int{
    "one": 1,
    "two": 2,
}

// Nil map (read-only, writes panic!)
var m map[string]int  // nil
```

### Basic Operations

```go
m := make(map[string]int)

// Insert/Update
m["key"] = 42

// Read (returns zero value if missing)
val := m["key"]

// Delete
delete(m, "key")

// Length
fmt.Println(len(m))
```

### Comma-Ok Idiom

```go
m := map[string]int{"a": 0}

// Without ok: Can't distinguish missing from zero
val := m["a"]   // 0
val := m["b"]   // 0 (missing, but same!)

// With ok: Proper detection
val, ok := m["a"]  // val=0, ok=true
val, ok := m["b"]  // val=0, ok=false

if v, ok := m["key"]; ok {
    fmt.Println("Found:", v)
} else {
    fmt.Println("Not found")
}
```

### Iteration

```go
for key, value := range m {
    fmt.Println(key, value)
}

// Order is RANDOM - changes each run!
```

### Maps with Struct Keys

```go
type Point struct{ X, Y int }

m := map[Point]string{
    {0, 0}: "origin",
    {1, 0}: "right",
}
```

## Common Pitfalls

1. **Writing to nil map panics**: `var m map[string]int; m["x"] = 1` crashes. Always initialize with `make` or a literal.

2. **Assuming iteration order**: Map iteration is intentionally randomized. Don't rely on any ordering.

3. **Concurrent map writes**: Concurrent reads are safe, but concurrent read+write causes runtime panic. Use `sync.Map` or mutex.

## Best Practices

- Always initialize maps before writing: `m := make(map[K]V)`.
- Use the comma-ok idiom when zero values are valid.
- Use `sync.Map` or `sync.RWMutex` for concurrent access.
- Pre-size with `make(map[K]V, n)` when size is known.

## Summary

Maps provide O(1) hash table operations with `m[key]` syntax. They're reference types that must be initialized before writing. The comma-ok idiom (`v, ok := m[key]`) distinguishes missing keys from zero values. Iteration order is randomized, and concurrent writes require synchronization.

## Code Examples

**Map Check**

```go
func main() {
    m := map[string]string{"foo": "bar", "baz": "qux"}
    
    if v, ok := m["foo"]; ok {
        fmt.Println("Found foo:", v)
    }
}
```


## Resources

- [Go Maps in Action](https://go.dev/blog/maps) — Official blog post on map usage
- [maps package](https://pkg.go.dev/maps) — Standard library map utilities

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*