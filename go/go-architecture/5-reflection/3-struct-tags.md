---
source_course: "go-architecture"
source_lesson: "go-architecture-struct-tags"
---

# Parsing Struct Tags

## Introduction
Struct tags are string metadata attached to struct fields, enclosed in backticks. They are the mechanism behind `json:"name"`, `db:"column"`, and `validate:"required"`. Learning to read and write custom struct tags lets you build validation frameworks, ORM mappers, and serialization libraries.

## Key Concepts
- **Struct Tag**: A raw string literal after a struct field's type, containing key-value pairs like `json:"name,omitempty"`.
- **Tag.Get()**: A method on `reflect.StructTag` that retrieves the value for a given key.
- **Custom Tags**: You can define your own tag keys (e.g., `validate:"required"`) and parse them with reflection.

## Real World Context
Validation libraries like `go-playground/validator` read struct tags to determine validation rules. ORM libraries read `db:"column_name"` tags to map struct fields to database columns. Understanding tag parsing lets you build similar tools or debug issues in existing ones.

## Deep Dive
Tags are attached to struct fields:

```go
type User struct {
    Name  string `json:"name" validate:"required"`
    Email string `json:"email" validate:"email"`
    Age   int    `json:"age,omitempty" validate:"gte=0"`
}
```

Reading tags with reflection uses `Tag.Get()`, which takes the tag key name and returns its value:

```go
t := reflect.TypeOf(User{})
field, ok := t.FieldByName("Name")
if ok {
    tag := field.Tag.Get("validate")
    fmt.Println(tag)  // "required"
}
```

Iterating all fields and their tags:

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

This outputs:
```
Name: json=name, validate=required
Email: json=email, validate=email
Age: json=age,omitempty, validate=gte=0
```

This is exactly how validation libraries and ORM mappers work internally.

## Common Pitfalls
1. **Malformed tag syntax** — Tags must follow the `key:"value"` format with proper quoting. Missing quotes or colons cause `Tag.Get()` to return empty strings.
2. **Forgetting to handle comma-separated values** — `json:"name,omitempty"` returns the full string. You must split on comma to extract individual parts.

## Best Practices
1. **Use standard tag key conventions** — Follow the `key:"value"` format. Common keys: `json`, `xml`, `db`, `validate`, `env`.
2. **Document your custom tags** — If you define custom tag keys, document the supported values and syntax.

## Summary
- Struct tags are string metadata on struct fields, read via reflection.
- `Tag.Get("key")` retrieves the value for a specific tag key.
- Tags power JSON serialization, ORM mapping, validation, and more.
- Always handle comma-separated tag values and malformed tags gracefully.

## Code Examples

**A helper that extracts the JSON field name from a struct tag, handling the common comma-separated format used for options like omitempty**

```go
// getJSONFieldName extracts the JSON field name from a struct tag.
// It handles the common "name,omitempty" format by splitting on comma.
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

// Usage: getJSONFieldName(reflect.TypeOf(User{}), "Age")
// Returns: "age"
```


## Resources

- [reflect.StructTag documentation](https://pkg.go.dev/reflect#StructTag) — API reference for StructTag including Get() and Lookup() methods
- [Go Wiki — Well-known struct tags](https://github.com/golang/go/wiki/Well-known-struct-tags) — Community-maintained list of commonly used struct tag keys

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*