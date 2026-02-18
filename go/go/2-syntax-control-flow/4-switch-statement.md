---
source_course: "go"
source_lesson: "go-switch-statement"
---

# Switch Statements

## Introduction

Switch statements provide elegant multi-way branching. Go's switch is safer and more powerful than its C ancestor—no automatic fallthrough means fewer bugs, and the ability to switch on any type (including strings) makes it incredibly versatile.

## Key Concepts

**Expression Switch**: Traditional switch comparing a value against multiple cases.

**Tagless Switch**: A switch without an expression, acting as a cleaner if-else chain.

**Fallthrough**: Explicit keyword to continue to the next case (opposite of C's default behavior).

**Type Switch**: A special form for examining the dynamic type of interface values.

## Real World Context

Switch statements frequently appear in HTTP handlers (switching on method or path), state machines, parsers, and any code with multiple discrete branches. The tagless switch pattern replaces complex if-else chains, improving readability. Type switches are essential when working with interface{} or any values from external sources.

## Deep Dive

### Basic Switch

```go
switch os := runtime.GOOS; os {
case "darwin":
    fmt.Println("macOS")
case "linux":
    fmt.Println("Linux")
case "windows":
    fmt.Println("Windows")
default:
    fmt.Println("Unknown:", os)
}
```

### Multiple Values per Case

```go
switch day {
case "Saturday", "Sunday":
    fmt.Println("Weekend!")
case "Monday", "Tuesday", "Wednesday", "Thursday", "Friday":
    fmt.Println("Weekday")
}
```

### Tagless Switch (if-else replacement)

```go
switch {
case hour < 12:
    return "Good morning"
case hour < 17:
    return "Good afternoon"
case hour < 21:
    return "Good evening"
default:
    return "Good night"
}
```

### Fallthrough (Explicit)

```go
switch n {
case 1:
    fmt.Println("One")
    fallthrough
case 2:
    fmt.Println("One or Two")
}
// Input 1 prints both lines
```

### Type Switch

```go
func describe(v any) {
    switch x := v.(type) {
    case int:
        fmt.Printf("Integer: %d\n", x)
    case string:
        fmt.Printf("String: %s\n", x)
    case bool:
        fmt.Printf("Boolean: %t\n", x)
    default:
        fmt.Printf("Unknown type: %T\n", x)
    }
}
```

## Common Pitfalls

1. **Expecting automatic fallthrough**: Coming from C, developers might expect cases to fall through. In Go, they don't—each case breaks automatically.

2. **Using fallthrough incorrectly**: `fallthrough` transfers control unconditionally to the next case's code, even if that case's condition wouldn't match.

3. **Duplicate case values**: The compiler catches identical case values as errors, but logic duplicates (overlapping ranges in tagless switch) aren't detected.

## Best Practices

- Prefer tagless switch over long if-else chains.
- Always include a `default` case for exhaustiveness.
- Avoid `fallthrough` unless truly necessary—it reduces code clarity.
- Use type switches when handling `any`/`interface{}` values.

## Summary

Go's switch statement supports expression matching, multiple values per case, and tagless form for if-else replacement. Cases don't fall through by default, eliminating a common source of bugs. Type switches enable elegant handling of interface values.

## Code Examples

**Switch without expression**

```go
func classify(n int) string {
    switch {
    case n < 0:
        return "negative"
    case n == 0:
        return "zero"
    case n < 10:
        return "small"
    default:
        return "large"
    }
}
```


## Resources

- [A Tour of Go - Switch](https://go.dev/tour/flowcontrol/9) — Interactive tour of switch statements
- [Effective Go - Switch](https://go.dev/doc/effective_go#switch) — Best practices for switch usage

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*