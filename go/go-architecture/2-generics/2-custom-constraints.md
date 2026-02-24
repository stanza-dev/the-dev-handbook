---
source_course: "go-architecture"
source_lesson: "go-architecture-custom-constraints"
---

# Custom Type Constraints

## Introduction
Built-in constraints like `any` and `comparable` cover basic cases, but real-world code often needs to restrict type parameters to specific sets of types or types with specific methods. Custom constraints give you this power. Understanding them unlocks advanced generic patterns.

## Key Concepts
- **Union Constraint**: An interface listing allowed types with `|` (e.g., `int | float64`).
- **Approximation Operator (~)**: The tilde prefix matches a type and all types with the same underlying type (e.g., `~int` matches `int` and `type MyInt int`).
- **Method Constraint**: An interface requiring specific methods, usable as a generic constraint.

## Real World Context
You are building a math library. You want a `Sum` function that works with `int`, `int64`, and `float64`, but not `string`. A union constraint restricts the type parameter to exactly the numeric types you support.

## Deep Dive
Union constraints list the allowed types:

```go
type Number interface {
    int | int64 | float64
}

func Sum[V Number](nums []V) V {
    var sum V
    for _, n := range nums {
        sum += n
    }
    return sum
}
```

Without the tilde, only exact types match. With `~`, defined types also match:

```go
type Integer interface {
    ~int | ~int64  // Includes type MyInt int
}

type MyInt int
var x MyInt = 42
// MyInt satisfies Integer because ~int matches types with int as underlying type
```

You can require methods in a constraint:

```go
type Stringer interface {
    String() string
}

func PrintAll[T Stringer](items []T) {
    for _, item := range items {
        fmt.Println(item.String())
    }
}
```

Constraints can combine type unions and methods:

```go
type OrderedStringer interface {
    cmp.Ordered
    fmt.Stringer
}
```

Go 1.26 introduces self-referential generic types, allowing patterns like:

```go
type Adder[A Adder[A]] interface {
    Add(A) A
}
```

This enables types to reference themselves in their own constraint, useful for mathematical abstractions and fluent APIs.

## Common Pitfalls
1. **Forgetting the tilde for custom types** — `int` alone won't match `type UserID int`. Use `~int` if you want to accept defined types.
2. **Mixing type elements and methods incorrectly** — A constraint with both type unions and methods is valid, but the type must satisfy both.

## Best Practices
1. **Use `~` by default for numeric constraints** — Most codebases define custom numeric types, and `~int` is more flexible.
2. **Compose constraints from smaller ones** — Just like interfaces, keep constraints focused and combine them when needed.

## Summary
- Union constraints (`int | float64`) restrict type parameters to specific types.
- The tilde `~` matches types with the same underlying type.
- Method constraints require specific methods on the type parameter.
- Constraints can combine type unions and method requirements.
- Go 1.26 adds self-referential generic types for advanced patterns.

## Code Examples

**A pointer method constraint pattern that allows generic code to call methods with pointer receivers — a common requirement for validation and mutation**

```go
// Constraint with method requirement using pointer receivers.
// This pattern lets you call Validate on pointer types generically.
type Validator[T any] interface {
    Validate() error
    *T  // T must have pointer receiver methods
}

func ValidateAll[T any, PT Validator[T]](items []T) error {
    for i := range items {
        if err := PT(&items[i]).Validate(); err != nil {
            return err
        }
    }
    return nil
}
```


## Resources

- [Go Specification — Type parameter declarations](https://go.dev/ref/spec#TypeConstraint) — Language specification for type constraints including union and tilde syntax
- [cmp package documentation](https://pkg.go.dev/cmp) — Standard library cmp package with Ordered constraint and comparison utilities

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*