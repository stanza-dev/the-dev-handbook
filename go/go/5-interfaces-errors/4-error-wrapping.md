---
source_course: "go"
source_lesson: "go-error-wrapping"
---

# Wrapping Errors (Go 1.13+)

Use `fmt.Errorf` with the `%w` verb to wrap errors, preserving the original error chain.

```go
if err != nil {
    return fmt.Errorf("failed to open file: %w", err)
}
```

## Inspecting Errors

### errors.Is

Check if an error matches a specific sentinel error:

```go
if errors.Is(err, os.ErrNotExist) {
    fmt.Println("File does not exist")
}
```

### errors.As

Extract a specific error type from the chain:

```go
var pathErr *os.PathError
if errors.As(err, &pathErr) {
    fmt.Println("Path:", pathErr.Path)
}
```

## Sentinel Errors

Define package-level errors for comparison:

```go
var (
    ErrNotFound = errors.New("not found")
    ErrInvalid  = errors.New("invalid")
)
```

Callers use `errors.Is(err, pkg.ErrNotFound)` to check.

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