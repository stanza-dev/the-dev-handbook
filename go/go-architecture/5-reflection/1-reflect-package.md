---
source_course: "go-architecture"
source_lesson: "go-architecture-reflect-package"
---

# Reflection

Reflection allows a program to inspect its own structure at runtime. It's how `json.Marshal` or `fmt.Println` work.

## Types and Values
*   `reflect.TypeOf(i)`: Returns the dynamic type.
*   `reflect.ValueOf(i)`: Returns the value.

```go
var x float64 = 3.14
t := reflect.TypeOf(x)
v := reflect.ValueOf(x)

fmt.Println(t)           // float64
fmt.Println(v)           // 3.14
fmt.Println(t.Kind())    // float64
fmt.Println(v.Float())   // 3.14
```

## Kind vs Type

*   **Type:** The specific type (e.g., `MyInt`).
*   **Kind:** The underlying kind (e.g., `int`).

```go
type MyInt int
var x MyInt = 42
t := reflect.TypeOf(x)
fmt.Println(t.Name())  // MyInt
fmt.Println(t.Kind())  // int
```

## Code Examples

**Inspecting Struct Fields**

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


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*