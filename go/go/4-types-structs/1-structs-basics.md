---
source_course: "go"
source_lesson: "go-structs-basics"
---

# Structs Fundamentals

## Introduction

Structs are Go's primary mechanism for creating custom types by grouping related data together. Unlike classes in OOP languages, Go structs are simple data containers—behavior is added separately through methods. This separation of data and behavior is a core Go philosophy.

## Key Concepts

**Struct**: A typed collection of fields, each with a name and type.

**Value Semantics**: Structs are values by default; assignment copies all fields.

**Exported Fields**: Fields starting with uppercase are visible outside the package.

**Anonymous Structs**: Structs defined inline without a named type.

## Real World Context

Structs model everything: database rows, API responses, configuration, domain entities. The exported/unexported distinction enables encapsulation without private/public keywords. Understanding struct value semantics prevents subtle bugs when modifying data.

## Deep Dive

### Declaration

```go
type User struct {
    ID        int
    Name      string
    Email     string
    CreatedAt time.Time
}
```

### Creating Structs

```go
// Zero value (all fields initialized)
var u User  // ID=0, Name="", etc.

// Struct literal (by field name - preferred)
u := User{
    ID:   1,
    Name: "Alice",
}

// Struct literal (by position - fragile)
u := User{1, "Alice", "a@example.com", time.Now()}

// Pointer to struct
p := &User{Name: "Bob"}
p := new(User)  // Returns *User with zero values
```

### Field Access

```go
u := User{Name: "Alice"}
u.Name = "Bob"  // Modify field

// Pointer access (auto-dereferenced)
p := &u
p.Name = "Charlie"  // Same as (*p).Name
```

### Exported vs Unexported

```go
type Account struct {
    ID       int    // Exported (visible outside package)
    Balance  int    // Exported
    password string // unexported (package-private)
}
```

### Anonymous/Inline Structs

```go
// Useful for one-off data structures
config := struct {
    Host string
    Port int
}{
    Host: "localhost",
    Port: 8080,
}
```

## Common Pitfalls

1. **Forgetting struct is a value**: Assigning `b := a` copies the entire struct. Modifying `b` doesn't affect `a` unless you use pointers.

2. **Positional literals breaking**: Adding a field to a struct breaks all positional literals. Always use named fields.

3. **Large struct copying**: Passing large structs by value copies all data. Use pointers for performance.

## Best Practices

- Always use named field literals: `User{Name: "Alice"}`.
- Use pointers to avoid copying large structs.
- Keep unexported fields for internal state; export only the API.
- Consider constructor functions for complex initialization.

## Summary

Structs group related fields into custom types. They have value semantics (assignment copies). Fields are exported if capitalized. Use struct literals with named fields for clarity, and pointers when you need to modify or avoid copying data.

## Code Examples

**Nested Structs**

```go
type Person struct {
    Name    string
    Age     int
    Address struct {
        City    string
        Country string
    }
}

func main() {
    p := Person{
        Name: "Alice",
        Age:  30,
    }
    p.Address.City = "Paris"
}
```


## Resources

- [A Tour of Go - Structs](https://go.dev/tour/moretypes/2) — Interactive struct tutorial
- [Effective Go - Data](https://go.dev/doc/effective_go#data) — Data structures and allocation

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*