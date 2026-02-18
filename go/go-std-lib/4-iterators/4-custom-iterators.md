---
source_course: "go-std-lib"
source_lesson: "go-std-lib-custom-iterators"
---

# Custom Iterator Patterns

## Filter

```go
func Filter[V any](seq iter.Seq[V], predicate func(V) bool) iter.Seq[V] {
    return func(yield func(V) bool) {
        for v := range seq {
            if predicate(v) {
                if !yield(v) {
                    return
                }
            }
        }
    }
}
```

## Map (Transform)

```go
func Map[V, R any](seq iter.Seq[V], transform func(V) R) iter.Seq[R] {
    return func(yield func(R) bool) {
        for v := range seq {
            if !yield(transform(v)) {
                return
            }
        }
    }
}
```

## Take

```go
func Take[V any](seq iter.Seq[V], n int) iter.Seq[V] {
    return func(yield func(V) bool) {
        count := 0
        for v := range seq {
            if count >= n {
                return
            }
            if !yield(v) {
                return
            }
            count++
        }
    }
}
```

## Chaining

```go
// Double even numbers, take first 3
result := slices.Collect(
    Take(
        Map(
            Filter(slices.Values(nums), isEven),
            double,
        ),
        3,
    ),
)
```

## Code Examples

**Infinite Iterator**

```go
// Generate infinite sequence
func Naturals() iter.Seq[int] {
    return func(yield func(int) bool) {
        for i := 1; ; i++ {
            if !yield(i) {
                return
            }
        }
    }
}

// Usage with Take
for n := range Take(Naturals(), 5) {
    fmt.Println(n)  // 1, 2, 3, 4, 5
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*