---
source_course: "go-std-lib"
source_lesson: "go-std-lib-json-encoding"
---

# JSON Encoding & Decoding

## Introduction

JSON is the lingua franca of web APIs, and Go makes working with it straightforward through the `encoding/json` package. Instead of writing parsers by hand, you define a Go struct and annotate it with struct tags that tell the JSON encoder how to map fields. If you have ever used Jackson in Java or Serde in Rust, Go's approach will feel refreshingly simple.

## Key Concepts

- **Struct Tags**: Metadata annotations (backtick strings) on struct fields that control JSON field names, omission rules, and more.
- **json.Marshal**: Converts a Go value into JSON bytes. Only exported (capitalized) fields are included.
- **json.Unmarshal**: Parses JSON bytes into a Go value. Missing fields get zero values; extra fields are silently ignored.
- **omitempty**: A tag option that omits a field from JSON output when it has its zero value (0, false, "", nil).

## Real World Context

Every REST API you build in Go will use `encoding/json`. When a frontend sends a POST request with a JSON body, you `Unmarshal` it into a struct, validate the fields, and process the data. When you send back a response, you `Marshal` a struct into JSON. Understanding struct tags and zero-value behavior is critical for building APIs that are both correct and ergonomic for your clients.

## Deep Dive

### Struct Tags

Struct tags tell the JSON package how to map Go field names to JSON keys. They are written as backtick-delimited strings after the field type.

```go
type User struct {
    Name string `json:"name"`
    Age  int    `json:"age,omitempty"` // Omit if zero value (0)
    Pass string `json:"-"`             // Never include in JSON
}
```

The `json:"name"` tag maps the `Name` field to the JSON key `"name"`. The `-` tag excludes a field entirely—useful for passwords and internal state. Without tags, the JSON package uses the Go field name as-is (`"Name"` instead of `"name"`).

### Marshalling (Go to JSON)

Converting a Go struct to JSON bytes is a single function call.

```go
user := User{Name: "Alice", Age: 30}
data, err := json.Marshal(user)
// data = []byte(`{"name":"Alice","age":30}`)
```

Because `Age` has a non-zero value, it is included. If `Age` were 0, the `omitempty` tag would cause it to be omitted from the output.

For human-readable output, use `MarshalIndent` which adds newlines and indentation.

```go
data, err := json.MarshalIndent(user, "", "  ")
// {
//   "name": "Alice",
//   "age": 30
// }
```

The second argument is a prefix for each line, and the third is the indentation string.

### Unmarshalling (JSON to Go)

Parsing JSON into a Go struct is equally simple. You pass a pointer to the target variable.

```go
data := []byte(`{"name": "Alice", "age": 30}`)
var user User
err := json.Unmarshal(data, &user)
if err != nil {
    log.Fatal(err)
}
fmt.Println(user.Name) // Alice
```

Fields present in the JSON but absent from the struct are silently ignored. Fields present in the struct but absent from the JSON get their zero value (0 for int, "" for string, nil for pointers).

### Working with Dynamic JSON

When you do not know the JSON structure ahead of time, you can unmarshal into a `map[string]any`.

```go
var result map[string]any
err := json.Unmarshal(data, &result)
name := result["name"].(string) // Type assertion required
```

This is flexible but loses type safety. Prefer defining structs when the schema is known.

### Pointer Fields and Null

Use pointer fields to distinguish between "field was absent" and "field was explicitly set to zero."

```go
type Update struct {
    Name *string `json:"name,omitempty"`
    Age  *int    `json:"age,omitempty"`
}
```

With pointer fields, a nil pointer means the field was not provided, while a pointer to a zero value means the client explicitly sent zero. This pattern is common in PATCH endpoints.

## Common Pitfalls

1. **Unexported fields are invisible to JSON** — A field named `name` (lowercase) cannot be marshalled or unmarshalled by `encoding/json`. You must capitalize it to `Name` and use a `json:"name"` tag if you want a lowercase JSON key.
2. **Confusing `omitempty` behavior with pointers** — `omitempty` omits nil pointers, but a pointer to a zero value (e.g., `*int` pointing to 0) is NOT omitted. The field is present because the pointer itself is non-nil.
3. **Assuming Unmarshal validates data** — `json.Unmarshal` does not check whether required fields are present or whether values are in valid ranges. You must validate separately after unmarshalling.

## Best Practices

1. **Always define struct tags for public API types** — Without tags, your JSON keys match Go naming conventions (`UserID` instead of `user_id`), which is non-idiomatic for most JSON consumers.
2. **Use `json.MarshalIndent` for debugging, `json.Marshal` for production** — Indented JSON is easier to read but uses more bandwidth. Use compact format in API responses.
3. **Use pointer fields for optional PATCH semantics** — This lets you distinguish between "field not provided" (nil) and "field set to zero value" (non-nil pointer to zero).

## Summary

- The `encoding/json` package uses struct tags to map Go fields to JSON keys.
- `json.Marshal` converts Go values to JSON bytes; `json.Unmarshal` parses JSON bytes into Go values.
- Only exported (capitalized) struct fields are visible to the JSON package.
- `omitempty` omits zero-value fields from JSON output; use pointer fields for nullable semantics.
- Unknown JSON fields are silently ignored during unmarshalling; missing fields get zero values.

## Code Examples

**Unmarshalling a JSON string into a Go struct — notice how field names must be exported (capitalized) and json tags map JSON keys to Go fields**

```go
data := []byte(`{"name": "Alice", "age": 30}`)
var u User
if err := json.Unmarshal(data, &u); err != nil {
    log.Fatal(err)
}
fmt.Println(u.Name)
```


## Resources

- [encoding/json package - Go Documentation](https://pkg.go.dev/encoding/json) — Official Go documentation for JSON encoding and decoding

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*