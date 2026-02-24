---
source_course: "go-architecture"
source_lesson: "go-architecture-reflect-package"
---

# The Reflect Package

## Introduction
Reflection allows a Go program to inspect and manipulate its own types and values at runtime. It is the mechanism behind `json.Marshal`, `fmt.Println`, and ORM libraries. While powerful, reflection trades compile-time safety for runtime flexibility. Understanding it is essential for building frameworks and understanding how Go's standard library works.

## Key Concepts
- **reflect.TypeOf()**: Returns the dynamic type of a value as a `reflect.Type`.
- **reflect.ValueOf()**: Returns the runtime value wrapped in a `reflect.Value`.
- **Kind vs Type**: Kind is the underlying category (int, struct, slice), while Type is the specific named type (MyInt, User).

## Real World Context
You are building a custom validation library that reads struct tags and validates field values. Without reflection, you would need to write validation logic for every struct individually. With reflection, you write it once and it works for any struct.

## Deep Dive
The two fundamental operations are getting a type and getting a value:

```go
var x float64 = 3.14
t := reflect.TypeOf(x)
v := reflect.ValueOf(x)

fmt.Println(t)           // float64
fmt.Println(v)           // 3.14
fmt.Println(t.Kind())    // float64
fmt.Println(v.Float())   // 3.14
```

Kind and Type are different concepts. Type is the specific named type, while Kind is the underlying category:

```go
type MyInt int
var x MyInt = 42
t := reflect.TypeOf(x)
fmt.Println(t.Name())  // MyInt    (the specific type)
fmt.Println(t.Kind())  // int      (the underlying kind)
```

You can inspect struct fields using NumField and Field:

```go
func printFields(x any) {
    v := reflect.ValueOf(x)
    t := v.Type()
    for i := 0; i < v.NumField(); i++ {
        fmt.Printf("Field: %s\tValue: %v\n",
            t.Field(i).Name, v.Field(i).Interface())
    }
}
```

Go 1.26 adds new iterator methods to `reflect.Type` and `reflect.Value`: `.Fields()`, `.Methods()`, `.Ins()`, `.Outs()`, and `reflect.Value.Fields()`, `.Methods()`. These return iterators and are more idiomatic than the old NumField/Field(i) loop pattern:

```go
// Go 1.26 idiomatic iteration
for field := range reflect.TypeOf(User{}).Fields() {
    fmt.Println(field.Name)
}
```

## Common Pitfalls
1. **Confusing Kind and Type** — `reflect.TypeOf(MyInt(42)).Kind()` is `int`, not `MyInt`. Use `.Name()` for the specific type.
2. **Calling NumField on non-struct types** — This panics. Always check `v.Kind() == reflect.Struct` first.

## Best Practices
1. **Check Kind before accessing type-specific methods** — Methods like NumField, Len, and MapKeys are only valid for specific Kinds.
2. **Use the new Go 1.26 iterator methods** — `.Fields()` and `.Methods()` are cleaner than manual index loops.

## Summary
- `reflect.TypeOf()` and `reflect.ValueOf()` are the entry points to reflection.
- Kind is the underlying category (int, struct); Type is the specific named type.
- Go 1.26 adds `.Fields()`, `.Methods()` iterator methods for idiomatic iteration.
- Always check Kind before calling type-specific reflection methods.

## Code Examples

**A generic struct field printer using reflection — it inspects any struct's fields and values at runtime without knowing the type at compile time**

```go
// printFields uses reflection to print all fields of any struct.
// It works with any struct type without knowing the fields at compile time.
func printFields(x any) {
    v := reflect.ValueOf(x)
    t := v.Type()

    for i := 0; i < v.NumField(); i++ {
        fmt.Printf("Field: %s\tValue: %v\n",
            t.Field(i).Name, v.Field(i).Interface())
    }
}

// Usage: printFields(User{Name: "Alice", Age: 30})
// Output:
// Field: Name  Value: Alice
// Field: Age   Value: 30
```


## Resources

- [Go Blog — The Laws of Reflection](https://go.dev/blog/laws-of-reflection) — The definitive article on Go's reflection model by Rob Pike
- [reflect package documentation](https://pkg.go.dev/reflect) — Complete API reference for the reflect package

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*