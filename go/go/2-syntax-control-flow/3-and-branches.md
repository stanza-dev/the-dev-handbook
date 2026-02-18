---
source_course: "go"
source_lesson: "go-loops-and-branches"
---

# Loops & Branching

## Introduction

Control flow determines which code runs and how many times. Go simplifies this by having just one looping construct (`for`) that handles all iteration patterns. Combined with `if/else` branching, you have everything needed to express complex logic clearly.

## Key Concepts

**For Loop**: Go's only loop construct, versatile enough to replace `while`, `do-while`, and traditional `for` loops from other languages.

**If Statement**: Conditional execution with optional initialization statement.

**Scoped Variables**: Variables declared in the `if` initialization are scoped to the entire if-else chain.

**Break/Continue**: Control statements for exiting loops or skipping iterations.

## Real World Context

Having a single loop construct means every Go developer reads loops the same way—no debates about when to use `while` vs `for`. The if-with-initialization pattern is heavily used for error checking (`if err := doThing(); err != nil`), creating a clean pattern that keeps error handling variables scoped appropriately.

## Deep Dive

### For Loop Variants

```go
// Classic three-component
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// While-style (condition only)
n := 0
for n < 5 {
    n++
}

// Infinite loop
for {
    if shouldStop() {
        break
    }
}

// Range over collections
nums := []int{1, 2, 3}
for index, value := range nums {
    fmt.Println(index, value)
}

// Range over string (iterates runes)
for i, r := range "Hello" {
    fmt.Printf("%d: %c\n", i, r)
}
```

### If Statements

```go
// Simple if
if x > 0 {
    return "positive"
}

// If with initialization (very common pattern)
if err := process(); err != nil {
    return err
}

// If-else chain
if x < 0 {
    return "negative"
} else if x == 0 {
    return "zero"
} else {
    return "positive"
}
```

### Loop Control

```go
for i := 0; i < 10; i++ {
    if i == 3 {
        continue  // Skip this iteration
    }
    if i == 7 {
        break     // Exit the loop
    }
}
```

## Common Pitfalls

1. **Forgetting the blank identifier**: When ranging, if you only need the value, use `for _, v := range items`. Unused variables cause compile errors.

2. **Modifying slice during range**: The range expression is evaluated once. Adding/removing elements during iteration can cause unexpected behavior.

3. **Variable shadowing in if initialization**: `if x := compute(); x > 0` creates a new `x` that shadows any outer `x` variable.

## Best Practices

- Use `for range` for iterating over slices, maps, and strings.
- Use the if-initialization pattern for error handling.
- Prefer early returns over deeply nested else blocks.
- Use `break` and `continue` with labeled loops for complex nested scenarios.

## Summary

Go uses `for` as its only loop keyword, supporting classic, while-style, infinite, and range-based iteration. If statements can include an initialization clause, scoping variables to the conditional block. The `break` and `continue` keywords control loop execution.

## Code Examples

**Basic For Loop**

```go
package main

import "fmt"

func main() {
    sum := 0
    for i := 0; i < 10; i++ {
        sum += i
    }
    fmt.Println(sum)
}
```


## Resources

- [Effective Go - Control Structures](https://go.dev/doc/effective_go#control-structures) — Best practices for loops and conditionals
- [A Tour of Go - For](https://go.dev/tour/flowcontrol/1) — Interactive tour of Go's for loop

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*