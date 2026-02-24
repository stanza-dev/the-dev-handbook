---
source_course: "go-performance"
source_lesson: "go-performance-cache-friendly-data"
---

# Cache-Friendly Data Structures

## Introduction
Modern CPUs don't access memory byte-by-byte — they load entire 64-byte cache lines at once. Data structures that keep related data contiguous in memory can be 10-100x faster than those that scatter data across the heap. This is the difference between array-of-structs and struct-of-arrays, and it profoundly affects Go program performance.

## Key Concepts
- **Cache line**: The smallest unit of data transferred between main memory and CPU cache. Typically 64 bytes on x86/ARM.
- **Spatial locality**: Accessing memory addresses close together. Sequential array access has excellent spatial locality.
- **Temporal locality**: Accessing the same memory address repeatedly in a short time period.
- **Struct of Arrays (SoA)**: Splitting a struct into parallel arrays — each field has its own array. Improves cache utilization when you only access some fields.

## Real World Context
A game engine processing 100,000 entities might only need their positions for collision detection. In an Array of Structs layout (each entity has all fields together), loading positions also pulls in health, name, and inventory data — wasting cache space. A Struct of Arrays layout keeps all positions contiguous, maximizing cache hits.

## Deep Dive

### Array of Structs (AoS) — Default Go Layout

```go
type Entity struct {
    X, Y, Z float64  // Position: 24 bytes
    Health   int32    // 4 bytes
    Name     string   // 16 bytes (header)
}

// All fields for each entity are contiguous
entities := make([]Entity, 100000)
```

When iterating over just positions, the CPU loads entire Entity structs into cache, wasting space on Health and Name.

### Struct of Arrays (SoA) — Cache-Friendly Alternative

```go
type Entities struct {
    X, Y, Z []float64  // Positions: contiguous
    Health   []int32    // Contiguous
    Name     []string   // Contiguous
}
```

Now iterating over positions only touches the X, Y, Z arrays — no wasted cache space.

### Benchmarking the Difference

```go
// AoS: process positions
for i := range entities {
    entities[i].X += velocity
}

// SoA: process positions — ~3-5x faster for large N
for i := range ents.X {
    ents.X[i] += velocity
}
```

The SoA version is faster because each cache line contains 8 float64 positions (64 bytes / 8 bytes = 8), while the AoS version wastes cache space loading Health and Name alongside each position.

### Linked Lists vs Slices

Linked lists have terrible cache performance because nodes are scattered across the heap. Each node traversal is a likely cache miss. Slices keep elements contiguous:

```go
// Bad: pointer chasing, random memory access
type Node struct { Value int; Next *Node }

// Good: contiguous, sequential access
values := make([]int, n)
```

Rule of thumb: prefer slices over linked structures unless you need O(1) insertion/removal at arbitrary positions.

## Common Pitfalls
1. **Using maps for ordered iteration** — Go maps have no guaranteed order and poor cache locality. Use sorted slices when you need ordered iteration over large datasets.
2. **Nested pointer structures** — A slice of pointers (`[]*Entity`) causes pointer chasing. Prefer a slice of values (`[]Entity`) when entities are small enough.

## Best Practices
1. **Use SoA for hot loops over specific fields** — When you process millions of elements but only touch 1-2 fields, SoA can be 3-10x faster.
2. **Keep hot data small** — The less data per iteration, the more iterations fit in a cache line.

## Summary
- CPUs load 64-byte cache lines — contiguous data access is dramatically faster.
- Struct of Arrays (SoA) layout improves cache utilization for field-specific iteration.
- Slices have far better cache performance than linked lists or pointer-heavy structures.
- Benchmark with realistic data sizes — cache effects only matter with enough data to exceed L1/L2 cache.

## Code Examples

**Struct of Arrays layout for particle simulation — position updates only touch position data, maximizing cache utilization**

```go
// Struct of Arrays — process positions without loading other fields
type Particles struct {
	X, Y, Z []float64 // Positions: contiguous in memory
	Mass    []float64 // Separate: only loaded when needed
	Color   []uint32  // Separate: only loaded for rendering
}

// Each cache line holds 8 positions (64 bytes / 8 bytes)
// vs AoS where cache lines are polluted with Mass and Color
func updatePositions(p *Particles, dt float64) {
	for i := range p.X {
		p.X[i] += p.X[i] * dt
		p.Y[i] += p.Y[i] * dt
		p.Z[i] += p.Z[i] * dt
	}
}
```


## Resources

- [Effective Go: Data Structures](https://go.dev/doc/effective_go) — Official Go guide covering data structure design and performance considerations

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*