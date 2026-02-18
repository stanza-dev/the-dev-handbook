---
source_course: "go"
source_lesson: "go-variables-and-types"
---

# Variables & Basic Types

## Introduction

Variables are the foundation of any program—they store the data your code manipulates. Go's variable system strikes a balance between the safety of static typing and the convenience of type inference, letting you write concise code without sacrificing compile-time checks.

## Key Concepts

**Static Typing**: Every variable has a fixed type determined at compile time, catching type errors before runtime.

**Type Inference**: The compiler can deduce the type from the assigned value, reducing verbosity.

**Zero Values**: Uninitialized variables receive a type-specific default value, eliminating undefined behavior.

**Constants**: Immutable values computed at compile time, providing type safety and optimization opportunities.

## Real World Context

Zero values eliminate an entire class of bugs common in C/C++ where uninitialized variables contain garbage. Static typing catches mismatched types during compilation rather than in production. The short declaration operator (`:=`) is used in virtually every Go function for cleaner, more readable code.

## Deep Dive

### Variable Declaration Styles

```go
// Explicit declaration with type
var name string = "Alice"
var age int = 30

// Type inference
var city = "Paris"  // string inferred

// Short declaration (inside functions only)
country := "France"  // Most common style

// Multiple declarations
var x, y, z int = 1, 2, 3
a, b := "hello", true
```

### Zero Values

| Type | Zero Value |
|------|------------|
| int, float64 | 0 |
| string | "" |
| bool | false |
| pointer, slice, map, channel, interface | nil |

```go
var count int      // 0
var name string    // ""
var active bool    // false
var data []byte    // nil
```

### Constants

```go
const Pi = 3.14159
const (
    StatusOK    = 200
    StatusError = 500
)

// iota for enumeration
const (
    Sunday = iota  // 0
    Monday         // 1
    Tuesday        // 2
)
```

## Common Pitfalls

1. **Using `:=` outside functions**: Short declarations only work inside function bodies. Use `var` at package level.

2. **Shadowing variables**: Declaring a new variable with the same name in an inner scope hides the outer one, leading to subtle bugs.

3. **Assuming nil slice equals empty slice**: A nil slice and an empty slice (`[]int{}`) behave differently in some contexts, though both have length 0.

## Best Practices

- Use `:=` inside functions for cleaner code.
- Use `var` for package-level variables or when you need the zero value explicitly.
- Group related constants with `const ()` blocks.
- Use `iota` for sequential constants instead of manual numbering.

## Summary

Go variables can be declared with `var` (explicit) or `:=` (short declaration inside functions). Type inference reduces boilerplate while maintaining static type safety. Zero values ensure variables are always initialized. Constants use `const` and are evaluated at compile time.

## Code Examples

**Variable Declarations**

```go
package main

import "fmt"

func main() {
    var a string = "initial"
    fmt.Println(a)

    b := 2
    fmt.Println(b)

    var c bool // defaults to false
    fmt.Println(c)
}
```


## Resources

- [A Tour of Go - Variables](https://go.dev/tour/basics/8) — Interactive tour covering Go variables
- [Effective Go - Declaration](https://go.dev/doc/effective_go#declaration) — Best practices for variable declarations

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*