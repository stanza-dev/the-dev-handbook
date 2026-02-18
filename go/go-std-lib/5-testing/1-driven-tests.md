---
source_course: "go-std-lib"
source_lesson: "go-std-lib-table-driven-tests"
---

# The Testing Package

Go has a built-in testing framework. Test files end in `_test.go` and functions start with `TestXxx(t *testing.T)`.

## Table-Driven Tests
The idiomatic way to test in Go is defining a slice of struct test cases.

```go
func TestAdd(t *testing.T) {
    tests := []struct{
        name string
        a, b, want int
    }{
        {"positive", 1, 2, 3},
        {"zero", 0, 0, 0},
        {"negative", -1, 1, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Add(tt.a, tt.b)
            if got != tt.want {
                t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.want)
            }
        })
    }
}
```

## Testing Methods

*   `t.Error(args...)`: Log and mark test as failed.
*   `t.Errorf(format, args...)`: Formatted error.
*   `t.Fatal(args...)`: Log and stop test immediately.
*   `t.Skip(args...)`: Skip this test.

## Code Examples

**Subtests with t.Run**

```go
func TestSplit(t *testing.T) {
    // ... define tests
    t.Run("subtest", func(t *testing.T) {
        // Subtests allow granular reporting
    })
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*