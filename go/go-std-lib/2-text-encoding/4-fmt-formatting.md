---
source_course: "go-std-lib"
source_lesson: "go-std-lib-fmt-formatting"
---

# The fmt Package

Go's `fmt` package provides powerful formatting.

## Printf Verbs

### General
*   `%v`: Default format.
*   `%+v`: With field names for structs.
*   `%#v`: Go syntax representation.
*   `%T`: Type of the value.

### Numbers
*   `%d`: Decimal integer.
*   `%x`: Hexadecimal.
*   `%f`: Float.
*   `%e`: Scientific notation.
*   `%.2f`: Float with 2 decimal places.

### Strings
*   `%s`: String.
*   `%q`: Quoted string.
*   `%c`: Character (rune).

## Sprintf for String Building

```go
msg := fmt.Sprintf("User %s has %d points", name, points)
```

## Errorf for Error Messages

```go
err := fmt.Errorf("failed to process item %d: %w", id, origErr)
```

## Fprintf to Any Writer

```go
fmt.Fprintf(os.Stderr, "Error: %v\n", err)
fmt.Fprintf(w, "Hello, %s!", name)  // HTTP response
```

## Code Examples

**Struct Formatting**

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


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*