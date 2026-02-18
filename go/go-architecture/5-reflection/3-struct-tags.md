---
source_course: "go-architecture"
source_lesson: "go-architecture-struct-tags"
---

# Struct Tags

Tags are string metadata attached to struct fields. `json:"..."` is just one example. You can define your own.

```go
type User struct {
    Name  string `json:"name" validate:"required"`
    Email string `json:"email" validate:"email"`
    Age   int    `json:"age,omitempty" validate:"gte=0"`
}
```

## Reading Tags

```go
t := reflect.TypeOf(User{})
field, ok := t.FieldByName("Name")
if ok {
    tag := field.Tag.Get("validate")
    fmt.Println(tag)  // "required"
}
```

## Iterating Fields and Tags

```go
t := reflect.TypeOf(User{})
for i := 0; i < t.NumField(); i++ {
    field := t.Field(i)
    fmt.Printf("%s: json=%s, validate=%s\n",
        field.Name,
        field.Tag.Get("json"),
        field.Tag.Get("validate"),
    )
}
```

This is how validation libraries (go-playground/validator) and ORM mappers work.

## Code Examples

**Parse JSON Tag**

```go
func getJSONFieldName(t reflect.Type, fieldName string) string {
    field, ok := t.FieldByName(fieldName)
    if !ok {
        return fieldName
    }
    jsonTag := field.Tag.Get("json")
    if jsonTag == "" || jsonTag == "-" {
        return fieldName
    }
    // Handle "name,omitempty"
    if idx := strings.Index(jsonTag, ","); idx != -1 {
        return jsonTag[:idx]
    }
    return jsonTag
}
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*