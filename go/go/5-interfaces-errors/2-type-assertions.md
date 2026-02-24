---
source_course: "go"
source_lesson: "go-empty-interface-type-assertions"
---

# Empty Interface & Type Assertions

## Introduction

The empty interface `interface{}` (aliased as `any` since Go 1.18) can hold a value of any type. While this gives you maximum flexibility, it sacrifices compile-time type safety. Type assertions and type switches let you recover the concrete type when you need it. Understanding these tools is essential for working with generic data, JSON parsing, and legacy APIs.

## Key Concepts

- **Empty Interface (`any`)**: An interface with no methods, satisfied by every type.
- **Type Assertion**: An expression that extracts the concrete type from an interface value.
- **Comma-Ok Pattern**: The safe form of type assertion that returns a boolean instead of panicking.
- **Type Switch**: A switch statement that branches on the dynamic type of an interface value.

## Real World Context

Before generics arrived in Go 1.18, `interface{}` was the only way to write functions accepting any type. You still encounter it in JSON unmarshaling (`map[string]any`), logging libraries, and event systems. Type assertions convert these untyped values back to concrete types for processing. Type switches are the idiomatic way to handle values that could be one of several types.

## Deep Dive

The empty interface has zero methods, which means every type satisfies it.

```go
var i any = "hello"
i = 42
i = []int{1, 2, 3}
```

The variable `i` can hold a string, an integer, or a slice because `any` imposes no method requirements.

### Type Assertions

A type assertion extracts the concrete value from an interface.

```go
var i any = "hello"

// Direct assertion (panics if wrong type)
s := i.(string)
```

The direct form panics at runtime if the type does not match. Always prefer the safe form in production code.

```go
// Safe assertion with ok check
s, ok := i.(string)
if !ok {
    fmt.Println("i is not a string")
}
```

The comma-ok pattern returns `false` instead of panicking, letting you handle the mismatch gracefully.

### Type Switches

For checking multiple types, a type switch is cleaner than chained type assertions.

```go
func describe(i any) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Integer: %d\n", v)
    case string:
        fmt.Printf("String: %s\n", v)
    case bool:
        fmt.Printf("Boolean: %t\n", v)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}
```

Inside each case, `v` is automatically typed to the matched type, giving you full access to its methods and fields.

## Common Pitfalls

1. **Using direct assertions without the ok check** — A failed direct assertion (`i.(string)` when `i` is an `int`) causes a runtime panic. Always use the comma-ok form unless you are certain of the type.
2. **Overusing `any` instead of specific interfaces** — Reaching for `any` when a small interface would suffice loses compile-time type safety. Define interfaces with the methods you need.

## Best Practices

1. **Prefer the comma-ok form for type assertions** — It prevents panics and makes error handling explicit.
2. **Use type switches over chained if-else assertions** — They are more readable and the compiler can optimize them better.

## Summary

- The empty interface `any` (formerly `interface{}`) can hold any value.
- Type assertions extract concrete types: use the comma-ok form to avoid panics.
- Type switches branch on the dynamic type cleanly and idiomatically.
- Prefer specific interfaces over `any` when possible for compile-time safety.
- Type assertions and switches are essential for JSON handling, logging, and generic data processing.

## Code Examples

**Type Switch**

```go
func process(data any) {
    switch v := data.(type) {
    case string:
        fmt.Println("String:", strings.ToUpper(v))
    case int:
        fmt.Println("Int doubled:", v*2)
    case []byte:
        fmt.Println("Bytes:", len(v))
    default:
        fmt.Printf("Unhandled: %T\n", v)
    }
}
```


## Resources

- [A Tour of Go - Type Assertions](https://go.dev/tour/methods/15) — Type assertions explained
- [A Tour of Go - Type Switches](https://go.dev/tour/methods/16) — Type switch syntax

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*