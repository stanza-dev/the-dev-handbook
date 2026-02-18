---
source_course: "go-performance"
source_lesson: "go-performance-cache-friendly-data"
---

# Data Locality

## Struct of Arrays vs Array of Structs

```go
// Array of Structs (AoS)
type Particle struct {
    X, Y, Z float64
    Mass    float64
}
particles := []Particle{...}

// Struct of Arrays (SoA)
type Particles struct {
    X, Y, Z []float64
    Mass    []float64
}
```

**When to use SoA:**
*   Processing one field at a time (better cache usage).
*   SIMD operations.
*   Large collections.

## Contiguous Memory

```go
// Bad: Pointer chasing
type Node struct {
    Value int
    Next  *Node
}

// Good: Contiguous slice
values := []int{1, 2, 3, 4, 5}
```

## Hot/Cold Splitting

```go
// Move rarely-accessed fields to separate struct
type User struct {
    ID   int64  // Hot
    Name string // Hot
}

type UserMeta struct {
    CreatedAt time.Time  // Cold
    Bio       string     // Cold
}
```

## Code Examples

**Struct of Arrays**

```go
// SoA for particle simulation
type Particles struct {
    X, Y, Z []float64
}

// Process X coordinates (cache-friendly)
func updateX(p *Particles, dt float64) {
    for i := range p.X {
        p.X[i] += dt  // Sequential memory access
    }
}
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*