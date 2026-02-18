---
source_course: "go-std-lib"
source_lesson: "go-std-lib-iter-slices-maps"
---

# Standard Library Iterators

Go 1.23 adds iterator functions to the `slices` and `maps` packages.

## slices Package

```go
import "slices"

// Iterate over values
for v := range slices.Values(mySlice) {
    fmt.Println(v)
}

// Backward iteration
for v := range slices.Backward(mySlice) {
    fmt.Println(v)
}

// Collect iterator into slice
result := slices.Collect(myIterator)
```

## maps Package

```go
import "maps"

// Iterate over keys
for k := range maps.Keys(myMap) {
    fmt.Println(k)
}

// Iterate over values
for v := range maps.Values(myMap) {
    fmt.Println(v)
}

// Iterate over key-value pairs
for k, v := range maps.All(myMap) {
    fmt.Printf("%s: %v\n", k, v)
}
```

## Creating Iterators from Collections

```go
// Convert slice to iterator
iter := slices.Values([]int{1, 2, 3})

// Convert iterator back to slice
slice := slices.Collect(iter)
```

## Code Examples

**Filtering with Iterators**

```go
import (
    "slices"
    "maps"
)

func main() {
    nums := []int{1, 2, 3, 4, 5}
    
    // Filter and collect
    evens := slices.Collect(
        Filter(slices.Values(nums), func(n int) bool {
            return n%2 == 0
        }),
    )
    fmt.Println(evens) // [2 4]
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*