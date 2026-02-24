---
source_course: "go-architecture"
source_lesson: "go-architecture-generics-basics"
---

# Type Parameters

## Introduction
Generics, introduced in Go 1.18, allow you to write functions and data structures that work with different types while maintaining full compile-time type safety. Before generics, you had to choose between code duplication and losing type safety with `interface{}`. Understanding type parameters is the foundation for writing reusable Go code.

## Key Concepts
- **Type Parameter**: A placeholder type declared in square brackets (e.g., `[T any]`) that gets replaced with a concrete type at compile time.
- **Constraint**: An interface that restricts which types a type parameter can accept.
- **Type Inference**: The compiler's ability to automatically determine type arguments from the values you pass.

## Real World Context
Without generics, writing a `Map` function that transforms a slice required either writing one version per type (`MapInt`, `MapString`) or accepting `[]interface{}` and losing type safety. Generics let you write it once, and the compiler catches type mismatches.

## Deep Dive
A generic function declares type parameters in square brackets before the regular parameters:

```go
func Map[T any, R any](s []T, f func(T) R) []R {
    result := make([]R, len(s))
    for i, v := range s {
        result[i] = f(v)
    }
    return result
}
```

The constraint after each type parameter controls what operations you can perform. The three most common built-in constraints are:

- `any`: Alias for `interface{}`, allows any type but only basic operations (assignment, comparison with reflect).
- `comparable`: Supports `==` and `!=`, required for map keys.
- `cmp.Ordered`: From the standard library `cmp` package (Go 1.21+), supports ordering operators `<`, `<=`, `>`, `>=`.

Here is a function using `cmp.Ordered`:

```go
import "cmp"

func Min[T cmp.Ordered](x, y T) T {
    if x < y {
        return x
    }
    return y
}
```

The compiler infers types when possible, so you can write `Min(3, 5)` instead of `Min[int](3, 5)`.

## Common Pitfalls
1. **Using `any` when you need ordering** — If your function uses `<` or `>`, you must constrain with `cmp.Ordered`, not `any`. The compiler will reject it.
2. **Importing the deprecated `golang.org/x/exp/constraints`** — Since Go 1.21, use the standard library `cmp` package instead. The `cmp.Ordered` constraint is built in.

## Best Practices
1. **Let the compiler infer types** — Only specify type arguments explicitly when inference fails or when it improves readability.
2. **Use the most specific constraint** — `comparable` is better than `any` when you need equality, and `cmp.Ordered` when you need ordering.

## Summary
- Type parameters let you write type-safe generic functions and types.
- Constraints (`any`, `comparable`, `cmp.Ordered`) control what operations are allowed.
- The `cmp` package in the standard library provides `Ordered` and other utilities.
- Type inference usually makes explicit type arguments unnecessary.

## Code Examples

**A generic Min function using the cmp.Ordered constraint from the standard library, demonstrating type inference with different types**

```go
import "cmp"

// Min returns the smaller of two ordered values.
// cmp.Ordered allows int, float64, string, and other ordered types.
func Min[T cmp.Ordered](x, y T) T {
    if x < y {
        return x
    }
    return y
}

// Usage:
// Min(3, 5)       // returns 3 (int inferred)
// Min("a", "b")   // returns "a" (string inferred)
```


## Resources

- [Tutorial: Getting started with generics](https://go.dev/doc/tutorial/generics) — Official Go tutorial introducing generic functions and type parameters
- [Go Blog — An Introduction To Generics](https://go.dev/blog/intro-generics) — In-depth blog post explaining the design and usage of Go generics

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*