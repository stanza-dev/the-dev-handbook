---
source_course: "go-architecture"
source_lesson: "go-architecture-custom-constraints"
---

# Custom Constraints

Constraints are interfaces that specify what types are allowed.

## Union Constraints

```go
type Number interface {
    int | int64 | float64
}

func Sum[V Number](nums []V) V {
    var sum V
    for _, n := range nums {
        sum += n
    }
    return sum
}
```

## Approximation (~)

The `~` allows types with the same underlying type:

```go
type Integer interface {
    ~int | ~int64  // Includes type MyInt int
}

type MyInt int
// MyInt satisfies Integer constraint
```

## Method Constraints

```go
type Stringer interface {
    String() string
}

func PrintAll[T Stringer](items []T) {
    for _, item := range items {
        fmt.Println(item.String())
    }
}
```

## Combining Constraints

```go
type OrderedStringer interface {
    constraints.Ordered
    fmt.Stringer
}
```

## Code Examples

**Pointer Method Constraint**

```go
// Constraint with method requirement
type Validator[T any] interface {
    Validate() error
    *T  // T must be pointer receiver
}

func ValidateAll[T any, PT Validator[T]](items []T) error {
    for i := range items {
        if err := PT(&items[i]).Validate(); err != nil {
            return err
        }
    }
    return nil
}
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*