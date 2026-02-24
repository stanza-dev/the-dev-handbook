---
source_course: "go-std-lib"
source_lesson: "go-std-lib-fmt-formatting"
---

# Formatted I/O with fmt

## Introduction

The `fmt` package is one of the first packages every Go developer encounters—`fmt.Println` is usually in your first Go program. But `fmt` goes far beyond simple printing. It provides a rich set of format verbs for precise control over how values are displayed, `Sprintf` for building strings, `Errorf` for creating structured errors, and `Fprintf` for writing to any `io.Writer`. Mastering these verbs makes your debugging output, log messages, and error reports significantly more useful.

## Key Concepts

- **Format verbs**: Placeholders like `%v`, `%s`, `%d` that control how values are formatted in the output string.
- **fmt.Sprintf**: Formats values into a string without printing, useful for building log messages and error strings.
- **fmt.Errorf**: Creates formatted error values, with `%w` support for wrapping errors in the error chain.
- **fmt.Fprintf**: Writes formatted output to any `io.Writer`, not just stdout.

## Real World Context

You are debugging a production issue where a struct has unexpected values. Printing with `%v` shows `{0 false}`, which is unhelpful. Switching to `%+v` shows `{UserID:0 Active:false}`, immediately revealing which fields are zeroed. In your error handling, `fmt.Errorf("failed to fetch user %d: %w", id, err)` creates an error that is both human-readable and programmatically inspectable with `errors.Is` and `errors.As`.

## Deep Dive

### General Verbs

The general verbs work with any type and are the most commonly used in day-to-day Go code.

```go
fmt.Printf("%v\n", user)   // Default format: {Alice 30}
fmt.Printf("%+v\n", user)  // With field names: {Name:Alice Age:30}
fmt.Printf("%#v\n", user)  // Go syntax: main.User{Name:"Alice", Age:30}
fmt.Printf("%T\n", user)   // Type: main.User
```

`%v` is the workhorse—it prints any value in a reasonable default format. `%+v` adds struct field names, which is invaluable for debugging. `%#v` shows the Go syntax you would write to create the value, including the type name. `%T` prints just the type.

### Number Verbs

For integers and floats, specific verbs control the base and precision.

```go
fmt.Printf("%d\n", 42)     // Decimal: 42
fmt.Printf("%x\n", 255)    // Hexadecimal: ff
fmt.Printf("%o\n", 8)      // Octal: 10
fmt.Printf("%b\n", 5)      // Binary: 101
fmt.Printf("%f\n", 3.14)   // Float: 3.140000
fmt.Printf("%.2f\n", 3.14) // 2 decimal places: 3.14
fmt.Printf("%e\n", 1000.0) // Scientific: 1.000000e+03
fmt.Printf("%9.2f\n", 3.14)// Right-aligned, width 9: "     3.14"
```

Width and precision modifiers (like `%9.2f`) give you columnar formatting for tables and reports. The width is the minimum total characters; the precision is decimal places for floats.

### String Verbs

String-specific verbs handle quoting, escaping, and raw bytes.

```go
fmt.Printf("%s\n", "hello")  // String: hello
fmt.Printf("%q\n", "hello")  // Quoted: "hello"
fmt.Printf("%x\n", "hello")  // Hex dump: 68656c6c6f
fmt.Printf("%c\n", 65)       // Character: A (from rune)
```

`%q` is particularly useful for logging user input because it escapes special characters, making invisible bytes like `\n` and `\t` visible in the output.

### Sprintf for String Building

`Sprintf` returns a formatted string instead of printing it. This is the standard way to build strings from mixed types.

```go
msg := fmt.Sprintf("User %s has %d points", name, points)
url := fmt.Sprintf("https://api.example.com/users/%d", id)
```

Unlike `strings.Builder`, `Sprintf` is not designed for loops—each call allocates a new string. Use it for one-shot string construction from templates.

### Errorf for Error Messages

`fmt.Errorf` creates error values with formatted messages. The `%w` verb wraps the original error, preserving the error chain.

```go
err := fmt.Errorf("failed to process item %d: %w", id, origErr)
```

The `%w` verb is special: it stores a reference to `origErr` so that `errors.Is(err, origErr)` returns `true` and `errors.As` can extract the original error type. This is essential for error handling in layered applications.

### Fprintf to Any Writer

`Fprintf` writes formatted output to any `io.Writer`, not just stdout. This makes it versatile for logging, HTTP responses, and file output.

```go
fmt.Fprintf(os.Stderr, "Error: %v\n", err)
fmt.Fprintf(w, "Hello, %s!", name)  // HTTP response
fmt.Fprintf(logFile, "[%s] %s\n", time.Now().Format(time.RFC3339), msg)
```

Because `Fprintf` works with the `io.Writer` interface, you can use it with files, network connections, buffers, and any other Writer.

## Common Pitfalls

1. **Using `%d` for a string or `%s` for an integer** — Go will print a warning like `%!d(string=hello)` instead of panicking. This silent misformat can hide bugs in log output. Use `go vet` to catch verb-type mismatches at compile time.
2. **Forgetting `%w` and using `%v` to wrap errors** — `%v` formats the error as a string but does NOT preserve the error chain. `errors.Is` and `errors.As` will not find the original error. Always use `%w` when wrapping errors.
3. **Using `Sprintf` in hot loops** — Each `Sprintf` call allocates a new string. For high-frequency logging or string building, use `strings.Builder` or pre-formatted constants.

## Best Practices

1. **Use `%+v` for debugging structs** — Field names in the output make it immediately clear what each value represents, especially for structs with multiple fields of the same type.
2. **Use `%w` consistently in error wrapping** — This enables `errors.Is` and `errors.As` to traverse the full error chain, which is critical for robust error handling.
3. **Run `go vet` to catch format string bugs** — The `go vet` tool detects mismatches between format verbs and argument types (e.g., `%d` with a string argument) at compile time.

## Summary

- `%v` is the default verb for any type; `%+v` adds field names; `%#v` shows Go syntax; `%T` shows the type.
- Number verbs include `%d` (decimal), `%x` (hex), `%f` (float), and `%.2f` (precision control).
- `fmt.Sprintf` builds formatted strings; `fmt.Errorf` with `%w` creates wrapped errors that preserve the error chain.
- `fmt.Fprintf` writes to any `io.Writer`, making it versatile for files, HTTP responses, and logging.
- Use `go vet` to catch verb-type mismatches before they cause silent formatting bugs.

## Code Examples

**Using %+v to print struct fields with names and %#v for Go syntax representation — useful for debugging complex data structures**

```go
type Point struct {
    X, Y int
}

p := Point{1, 2}
fmt.Printf("%v\n", p)   // {1 2}
fmt.Printf("%+v\n", p)  // {X:1 Y:2}
fmt.Printf("%#v\n", p)  // main.Point{X:1, Y:2}
fmt.Printf("%T\n", p)   // main.Point
```


## Resources

- [fmt package - Go Documentation](https://pkg.go.dev/fmt) — Official Go documentation for formatted I/O with Printf, Sprintf, and verbs

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*