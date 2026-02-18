---
source_course: "go"
source_lesson: "go-error-handling"
---

# Errors are Values

Go does not have exceptions. Errors are just values returned by functions.

```go
f, err := os.Open("filename.ext")
if err != nil {
    log.Fatal(err)
}
```

## The error Interface
```go
type error interface {
    Error() string
}
```

## Creating Errors

```go
import "errors"

err := errors.New("something went wrong")

// With formatting
import "fmt"
err := fmt.Errorf("failed to process item %d", id)
```

## Custom Error Types

```go
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}
```

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