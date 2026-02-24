---
source_course: "go-architecture"
source_lesson: "go-architecture-reflect-performance"
---

# Reflection Performance & Best Practices

## Introduction
Reflection is powerful but comes with significant performance overhead and loss of type safety. Knowing when to use reflection—and when to avoid it—is crucial for building performant Go applications. This lesson covers the trade-offs, caching strategies, and guidelines for responsible reflection use.

## Key Concepts
- **Performance Overhead**: Reflection is roughly 100x slower than direct field access due to runtime type lookups and allocations.
- **Type Caching**: Storing computed type information in a map to avoid repeated reflection on the same types.
- **Alternatives**: Generics, type switches, and code generation can often replace reflection with better performance.

## Real World Context
You are writing a JSON serialization library. The first call to `Marshal(User{})` must use reflection to discover User's fields. But subsequent calls should not repeat this work. Caching the field metadata in a `sync.Map` or `RWMutex`-protected map gives you reflection's flexibility with near-native performance after the first call.

## Deep Dive
Reflection has measurable overhead:

```go
func BenchmarkDirect(b *testing.B) {
    u := User{Name: "Alice"}
    for i := 0; i < b.N; i++ {
        _ = u.Name  // ~0.3ns
    }
}

func BenchmarkReflect(b *testing.B) {
    u := User{Name: "Alice"}
    v := reflect.ValueOf(u)
    for i := 0; i < b.N; i++ {
        _ = v.FieldByName("Name").String()  // ~30ns
    }
}
```

Cache type information to reduce overhead:

```go
var typeCache = make(map[reflect.Type][]fieldInfo)
var typeCacheMu sync.RWMutex

func getCachedFields(t reflect.Type) []fieldInfo {
    typeCacheMu.RLock()
    if fields, ok := typeCache[t]; ok {
        typeCacheMu.RUnlock()
        return fields
    }
    typeCacheMu.RUnlock()

    fields := computeFields(t)
    typeCacheMu.Lock()
    typeCache[t] = fields
    typeCacheMu.Unlock()
    return fields
}
```

Valid use cases for reflection include: serialization (JSON, XML), ORM field mapping, generic validators, struct tag parsing, and testing utilities. Avoid reflection in hot code paths, when generics can solve the problem, and when simple type switches work.

Go 1.26's iterator methods (`.Fields()`, `.Methods()`) make reflection code cleaner but do not change the performance characteristics. Always cache when iterating types repeatedly.

## Common Pitfalls
1. **Using reflection in hot loops** — Reflection in tight loops can be a 100x performance regression. Cache type info or use code generation.
2. **Choosing reflection over generics** — Since Go 1.18, many use cases that required reflection can be handled with generics at compile time.

## Best Practices
1. **Cache type metadata** — Use a `sync.RWMutex`-protected map or `sync.Map` to store computed field info per type.
2. **Prefer generics for type-safe operations** — If your logic does not need runtime type discovery, generics are faster and safer.

## Summary
- Reflection is ~100x slower than direct access and allocates memory.
- Cache type information to amortize the cost of reflection.
- Valid use cases: serialization, ORM, validation, tag parsing.
- Avoid reflection in hot paths; prefer generics or code generation.

## Code Examples

**Benchmark comparing direct field access (~0.3ns) versus reflection-based access (~30ns), demonstrating the ~100x performance overhead of reflection**

```go
// Benchmark showing the performance difference between
// direct field access and reflection-based access.
func BenchmarkDirect(b *testing.B) {
    u := User{Name: "Alice"}
    for i := 0; i < b.N; i++ {
        _ = u.Name  // ~0.3ns per operation
    }
}

func BenchmarkReflect(b *testing.B) {
    u := User{Name: "Alice"}
    v := reflect.ValueOf(u)
    for i := 0; i < b.N; i++ {
        _ = v.FieldByName("Name").String()  // ~30ns per operation
    }
}
```


## Resources

- [Go Blog — The Laws of Reflection](https://go.dev/blog/laws-of-reflection) — Foundational article on Go reflection by Rob Pike
- [Go 1.26 Release Notes](https://go.dev/doc/go1.26) — Release notes covering new reflect iterator methods in Go 1.26

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*