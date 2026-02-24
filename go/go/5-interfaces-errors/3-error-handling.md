---
source_course: "go"
source_lesson: "go-error-handling"
---

# Error Handling

## Introduction

Go takes a radically different approach to errors compared to languages with exceptions. In Go, errors are values, returned as the last return value from functions. This makes error handling explicit, visible, and impossible to accidentally ignore. The pattern `if err != nil` is the heartbeat of Go code.

## Key Concepts

- **Error as Value**: Errors are regular values implementing the `error` interface, not special control flow constructs.
- **The `error` Interface**: A built-in interface with a single `Error() string` method.
- **Sentinel Errors**: Package-level error variables used for comparison (e.g., `io.EOF`).
- **Custom Error Types**: Structs implementing the `error` interface that carry additional context.

## Real World Context

Explicit error handling is one of Go's most debated features, but it provides enormous value in production. Every error path is visible in the code, making it easy to audit and test. Unlike exceptions, errors cannot silently propagate through call stacks. Teams at Google, Cloudflare, and Docker cite explicit error handling as a key reason their Go services are reliable.

## Deep Dive

Functions that can fail return an error as their last value. The caller checks it immediately.

```go
f, err := os.Open("filename.ext")
if err != nil {
    log.Fatal(err)
}
```

This pattern appears on nearly every line that calls a fallible function.

### The error Interface

The built-in `error` interface is minimal by design.

```go
type error interface {
    Error() string
}
```

Any type with an `Error() string` method satisfies it.

### Creating Errors

The standard library provides two common ways to create errors.

```go
import "errors"

err := errors.New("something went wrong")

import "fmt"
err := fmt.Errorf("failed to process item %d", id)
```

Use `errors.New` for static messages and `fmt.Errorf` when you need formatting.

### Custom Error Types

For errors that carry structured data, define a custom type.

```go
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}
```

Callers can use type assertions or `errors.As` to extract the structured data.

## Common Pitfalls

1. **Ignoring returned errors** — Using `_` to discard an error (e.g., `result, _ := doThing()`) hides failures that may corrupt state later.
2. **Returning a concrete nil pointer as an interface** — Returning `(*MyError)(nil)` as `error` creates a non-nil interface, surprising callers who check `err != nil`.

## Best Practices

1. **Always check errors immediately** — Handle or return every error right after the call that produces it.
2. **Add context when propagating** — Use `fmt.Errorf("doing X: %w", err)` so the final error message tells a complete story.

## Summary

- Go treats errors as values, not exceptions.
- The `error` interface requires only an `Error() string` method.
- Create errors with `errors.New` or `fmt.Errorf`.
- Custom error types carry structured context for callers.
- Always check and handle errors immediately after the call.

## Code Examples

**Returning Errors**

```go
import "errors"

func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```


## Resources

- [A Tour of Go - Errors](https://go.dev/tour/methods/19) — Introduction to error handling
- [Effective Go - Errors](https://go.dev/doc/effective_go#errors) — Error handling best practices

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*