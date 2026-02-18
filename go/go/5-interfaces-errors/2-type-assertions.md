---
source_course: "go"
source_lesson: "go-empty-interface-type-assertions"
---

# The Empty Interface

`interface{}` (or `any` in Go 1.18+) creates a variable that can hold values of any type.

```go
var i any = "hello"
i = 42
i = []int{1, 2, 3}
```

## Type Assertions

To get the concrete value back, use a **type assertion**:

```go
var i any = "hello"

// Direct assertion (panics if wrong type)
s := i.(string)

// Safe assertion with ok check
s, ok := i.(string)
if !ok {
    fmt.Println("i is not a string")
}
```

## Type Switches

For multiple type checks, use a type switch:

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