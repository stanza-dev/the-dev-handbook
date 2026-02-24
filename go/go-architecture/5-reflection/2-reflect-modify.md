---
source_course: "go-architecture"
source_lesson: "go-architecture-reflect-modify"
---

# Modifying Values with Reflection

## Introduction
Reading values with reflection is straightforward, but modifying them requires understanding addressability. A reflected value can only be set if it points to the original variable's memory. This is one of the most common sources of confusion (and panics) when working with reflection.

## Key Concepts
- **Addressability**: A reflect.Value is settable only if it was obtained from a pointer. `reflect.ValueOf(x)` copies x; `reflect.ValueOf(&x).Elem()` points to x.
- **CanSet()**: Returns true if the value can be modified. Always check this before calling Set methods.
- **Elem()**: Dereferences a pointer or interface Value to get the underlying value.

## Real World Context
You are building a configuration loader that reads environment variables and sets struct fields by name. Each field must be settable, which means you must pass a pointer to the struct. Understanding addressability is what makes this work.

## Deep Dive
This code panics because `reflect.ValueOf(x)` creates a copy:

```go
x := 1
v := reflect.ValueOf(x)
v.SetInt(2)  // PANIC: unaddressable value
```

The fix is to pass a pointer and call Elem():

```go
x := 1
v := reflect.ValueOf(&x).Elem()  // Get addressable value
v.SetInt(2)  // Works!
fmt.Println(x)  // 2
```

Always check settability before modifying:

```go
v := reflect.ValueOf(x)
if v.CanSet() {
    v.SetInt(2)
}
```

For structs, modify fields by name:

```go
type User struct {
    Name string
    Age  int
}

u := User{Name: "Alice", Age: 30}
v := reflect.ValueOf(&u).Elem()
v.FieldByName("Name").SetString("Bob")
v.FieldByName("Age").SetInt(25)
```

Only exported fields can be modified via reflection. Attempting to set an unexported field panics.

## Common Pitfalls
1. **Forgetting to pass a pointer** — `reflect.ValueOf(x)` copies x, making the Value unaddressable. Always use `reflect.ValueOf(&x).Elem()`.
2. **Setting unexported fields** — This panics. Only exported (capitalized) fields are settable via reflection.

## Best Practices
1. **Always check CanSet()** — Before calling any Set method, verify the value is settable to avoid panics.
2. **Check IsValid() for FieldByName** — `FieldByName("nonexistent")` returns an invalid Value. Check `.IsValid()` before using it.

## Summary
- `reflect.ValueOf(x)` copies x and is not settable. Use `reflect.ValueOf(&x).Elem()` instead.
- Always check `CanSet()` before modifying values.
- Only exported struct fields can be modified via reflection.
- Use `FieldByName()` to access struct fields dynamically, checking `IsValid()` first.

## Code Examples

**A generic field setter that modifies any struct field by name using reflection — it validates the pointer requirement and field settability before modification**

```go
// setField sets a struct field by name using reflection.
// The obj parameter must be a pointer to a struct.
func setField(obj any, name string, value any) error {
    v := reflect.ValueOf(obj)
    if v.Kind() != reflect.Ptr {
        return errors.New("must pass pointer")
    }

    field := v.Elem().FieldByName(name)
    if !field.IsValid() {
        return fmt.Errorf("no field %s", name)
    }
    if !field.CanSet() {
        return fmt.Errorf("cannot set %s (unexported?)", name)
    }

    field.Set(reflect.ValueOf(value))
    return nil
}

// Usage: setField(&user, "Name", "Bob")
```


## Resources

- [Go Blog — The Laws of Reflection](https://go.dev/blog/laws-of-reflection) — Covers the three laws of reflection including settability

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*