---
source_course: "go-architecture"
source_lesson: "go-architecture-reflect-performance"
---

# Reflection Trade-offs

## Performance

Reflection is slow:
*   ~100x slower than direct access.
*   Allocates memory.
*   No compile-time type checking.

## When to Use

*   Serialization (JSON, XML, etc.).
*   ORM field mapping.
*   Generic validators.
*   Testing utilities.

## When NOT to Use

*   Hot code paths.
*   When generics can solve it.
*   When type switches work.

## Caching

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
    
    // Compute and cache
    fields := computeFields(t)
    typeCacheMu.Lock()
    typeCache[t] = fields
    typeCacheMu.Unlock()
    return fields
}
```

## Code Examples

**Reflection Benchmark**

```go
// Benchmark showing reflection overhead
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


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*