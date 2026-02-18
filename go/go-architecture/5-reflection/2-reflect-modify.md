---
source_course: "go-architecture"
source_lesson: "go-architecture-reflect-modify"
---

# Modifying Values

To modify a value via reflection, you must pass a pointer and get the element.

## The Problem

```go
x := 1
v := reflect.ValueOf(x)
v.SetInt(2)  // PANIC: unaddressable value
```

## The Solution

```go
x := 1
v := reflect.ValueOf(&x).Elem()  // Get addressable value
v.SetInt(2)  // Works!
fmt.Println(x)  // 2
```

## Checking Settability

```go
v := reflect.ValueOf(x)
if v.CanSet() {
    v.SetInt(2)
}
```

## Modifying Struct Fields

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

**Note:** Only exported fields can be modified.

## Code Examples

**Generic Field Setter**

```go
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
        return fmt.Errorf("cannot set %s", name)
    }
    
    field.Set(reflect.ValueOf(value))
    return nil
}
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*