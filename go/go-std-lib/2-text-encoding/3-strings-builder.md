---
source_course: "go-std-lib"
source_lesson: "go-std-lib-strings-builder"
---

# Immutable Strings

Strings in Go are immutable. Doing `s += "a"` creates a new string and copies data, which is O(N²) in a loop.

## strings.Builder
Use `strings.Builder` to concatenate strings efficiently (O(N)).

```go
var sb strings.Builder
for i := 0; i < 1000; i++ {
    sb.WriteString("hello")
}
res := sb.String()
```

## Builder Methods

*   `WriteString(s)`: Append a string.
*   `WriteByte(b)`: Append a single byte.
*   `WriteRune(r)`: Append a rune.
*   `String()`: Get the final string.
*   `Reset()`: Clear and reuse.

## strings Package Functions

```go
strings.Contains(s, "needle")     // Check substring
strings.HasPrefix(s, "Go")        // Check prefix
strings.Split(s, ",")             // Split into slice
strings.Join(slice, ", ")         // Join slice
strings.ToLower(s)                // Lowercase
strings.TrimSpace(s)              // Remove whitespace
strings.ReplaceAll(s, "old", "new")
```

## Code Examples

**String Builder Example**

```go
import "strings"

func join(strs []string) string {
    var sb strings.Builder
    for _, s := range strs {
        sb.WriteString(s)
    }
    return sb.String()
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*