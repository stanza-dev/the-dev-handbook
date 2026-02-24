---
source_course: "go-architecture"
source_lesson: "go-architecture-generics-best-practices"
---

# Generics Best Practices

## Introduction
Generics are powerful but not always the right tool. Knowing when to use generics—and when to stick with interfaces or concrete types—is a critical skill. This lesson covers the guidelines that help you write clear, maintainable generic code.

## Key Concepts
- **Type Inference**: The compiler deduces type arguments from values, so you rarely need to specify them explicitly.
- **Zero Value**: `var zero T` produces the zero value for any type parameter.
- **Constraint Selection**: Choose the minimum constraint that enables the operations you need.

## Real World Context
You are reviewing a pull request that converts a simple `ToString(v any) string` function into `ToString[T any](v T) string`. The generic version adds complexity without benefit—`any` already accepts all types. Knowing when generics add value versus noise saves your team from over-engineering.

## Deep Dive
Generics shine for data structures and algorithms:

```go
// Good use: generic utility function
func Filter[T any](s []T, f func(T) bool) []T {
    result := make([]T, 0)
    for _, v := range s {
        if f(v) {
            result = append(result, v)
        }
    }
    return result
}

evens := Filter(nums, func(n int) bool { return n%2 == 0 })
```

But they add noise when concrete types or `any` suffice:

```go
// Bad: unnecessary generic
func ToString[T any](v T) string {
    return fmt.Sprintf("%v", v)
}

// Good: just use any
func ToString(v any) string {
    return fmt.Sprintf("%v", v)
}
```

Good use cases include: data structures (stacks, queues, trees, sets), utility functions (Map, Filter, Reduce), and algorithms (sort, search, min/max). Avoid generics when: a concrete type works fine, only 2-3 types are involved (separate functions may be clearer), or behavior varies by type (use interfaces instead).

Let the compiler infer types whenever possible:

```go
// Verbose (unnecessary)
result := Map[int, string](nums, strconv.Itoa)

// Clean (compiler infers int, string)
result := Map(nums, strconv.Itoa)
```

Self-referential generic types (Go 1.26) enable advanced patterns:

```go
type Adder[A Adder[A]] interface {
    Add(A) A
}
```

This constrains A to be a type that can add itself, useful for mathematical abstractions and builder patterns.

## Common Pitfalls
1. **Over-genericizing simple functions** — If a function only uses `fmt.Sprintf("%v", v)`, it does not benefit from generics. Use `any` directly.
2. **Ignoring type inference** — Specifying type arguments explicitly when the compiler can infer them adds visual noise.

## Best Practices
1. **Start with concrete types** — Only introduce generics when you find yourself duplicating the same logic for different types.
2. **Use the narrowest constraint** — `comparable` is better than `any` when you need equality. `cmp.Ordered` when you need ordering.

## Summary
- Use generics for data structures, utility functions, and algorithms.
- Avoid generics when concrete types or `any` suffice.
- Let the compiler infer type arguments whenever possible.
- Start concrete, refactor to generic only when duplication appears.
- Go 1.26 adds self-referential generic types for advanced use cases.

## Code Examples

**A generic Filter function that works with any slice type — type inference means callers never need to specify the type argument**

```go
// Good generic use case: a reusable Filter function.
// The type parameter T is inferred from the slice type.
func Filter[T any](s []T, f func(T) bool) []T {
    result := make([]T, 0)
    for _, v := range s {
        if f(v) {
            result = append(result, v)
        }
    }
    return result
}

// Usage: evens := Filter(nums, func(n int) bool { return n%2 == 0 })
```


## Resources

- [Go Blog — When To Use Generics](https://go.dev/blog/when-generics) — Official guidance on when generics improve code versus when they add unnecessary complexity
- [Go 1.26 Release Notes](https://go.dev/doc/go1.26) — Release notes covering new language features including self-referential generic types

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*