---
source_course: "go"
source_lesson: "go-error-wrapping"
---

# Error Wrapping & Inspection

## Introduction

When an error travels up through multiple layers of a program, context gets lost. Error wrapping (introduced in Go 1.13) solves this by letting you add context to an error while preserving the original for later inspection. Combined with `errors.Is` and `errors.As`, wrapping creates rich, inspectable error chains.

## Key Concepts

- **Error Wrapping**: Adding context to an error using `fmt.Errorf` with the `%w` verb, preserving the original error.
- **Error Chain**: A linked sequence of wrapped errors from the outermost context to the root cause.
- **`errors.Is`**: Checks if any error in the chain matches a specific sentinel value.
- **`errors.As`**: Extracts a specific error type from anywhere in the chain.

## Real World Context

In production services, a database timeout might propagate through a repository layer, a service layer, and an HTTP handler. Without wrapping, you see only "connection timed out." With wrapping, you see "handling request: fetching user: querying database: connection timed out"—a complete trace. Teams use `errors.Is` to make decisions (retry on timeout?) and `errors.As` to extract structured data (which field failed validation?).

## Deep Dive

The `%w` verb in `fmt.Errorf` wraps an error, creating a chain.

```go
if err != nil {
    return fmt.Errorf("failed to open file: %w", err)
}
```

The returned error's message includes the context, and the original error is preserved for inspection.

### Inspecting with errors.Is

`errors.Is` traverses the entire error chain looking for a match against a sentinel error.

```go
if errors.Is(err, os.ErrNotExist) {
    fmt.Println("File does not exist")
}
```

This works even if the error has been wrapped multiple times.

### Inspecting with errors.As

`errors.As` finds the first error in the chain that matches a target type and extracts it.

```go
var pathErr *os.PathError
if errors.As(err, &pathErr) {
    fmt.Println("Path:", pathErr.Path)
}
```

This lets you access structured fields from custom error types deep in the chain.

### Go 1.26: errors.AsType[T]()

Go 1.26 introduces `errors.AsType[T]()` as a generic alternative to `errors.As`. Instead of declaring a variable and passing its pointer, you get the typed error directly.

```go
// Traditional errors.As
var pathErr *os.PathError
if errors.As(err, &pathErr) { ... }

// Go 1.26 generic alternative
if pathErr, ok := errors.AsType[*os.PathError](err); ok { ... }
```

The generic form is more concise and avoids the need for a separate variable declaration.

### Sentinel Errors

Define package-level errors for callers to compare against.

```go
var (
    ErrNotFound = errors.New("not found")
    ErrInvalid  = errors.New("invalid")
)
```

Callers use `errors.Is(err, pkg.ErrNotFound)` to check for specific conditions regardless of wrapping.

## Common Pitfalls

1. **Using `%v` instead of `%w`** — `fmt.Errorf("context: %v", err)` converts the error to a string, breaking the chain. `errors.Is` and `errors.As` will no longer find the original error.
2. **Wrapping errors you should not expose** — Wrapping preserves the original error type. If you wrap an internal database error, callers can inspect it with `errors.As`, potentially leaking implementation details.

## Best Practices

1. **Always use `%w` when you want callers to inspect the cause** — This preserves the chain for `errors.Is` and `errors.As`.
2. **Add meaningful context at each layer** — Each wrap should explain what the current function was trying to do, building a readable error narrative.

## Summary

- Wrap errors with `fmt.Errorf("context: %w", err)` to preserve the chain.
- `errors.Is` checks for sentinel error matches anywhere in the chain.
- `errors.As` extracts a specific error type from the chain for structured inspection.
- Use `%w` (not `%v`) to maintain the error chain.
- Define sentinel errors at the package level for stable comparison targets.

## Code Examples

**Error Wrapping Pattern**

```go
func readConfig(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("reading config: %w", err)
    }
    
    var cfg Config
    if err := json.Unmarshal(data, &cfg); err != nil {
        return nil, fmt.Errorf("parsing config: %w", err)
    }
    
    return &cfg, nil
}
```


## Resources

- [Go Blog - Error handling in Go 1.13](https://go.dev/blog/go1.13-errors) — Introduction to error wrapping
- [errors package](https://pkg.go.dev/errors) — Standard library error utilities

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*