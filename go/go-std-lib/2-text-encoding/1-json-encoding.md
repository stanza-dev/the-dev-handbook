---
source_course: "go-std-lib"
source_lesson: "go-std-lib-json-encoding"
---

# encoding/json

Go maps JSON objects to Structs using **struct tags**.

```go
type User struct {
    Name string `json:"name"`
    Age  int    `json:"age,omitempty"` // Omit if zero value (0)
    Pass string `json:"-"`             // Ignore this field
}
```

## Marshalling & Unmarshalling
*   `json.Marshal(v)`: Struct -> JSON bytes.
*   `json.Unmarshal(data, &v)`: JSON bytes -> Struct.

```go
user := User{Name: "Alice", Age: 30}
data, err := json.Marshal(user)
// data = []byte(`{"name":"Alice","age":30}`)

var decoded User
err = json.Unmarshal(data, &decoded)
```

## Important Notes

*   Fields must be **Exported** (Capitalized) to be visible to the JSON package.
*   Use `json.MarshalIndent` for pretty printing.
*   `omitempty` works for zero values: 0, false, "", nil.

## Code Examples

**Unmarshalling JSON**

```go
data := []byte(`{"name": "Alice", "age": 30}`)
var u User
if err := json.Unmarshal(data, &u); err != nil {
    log.Fatal(err)
}
fmt.Println(u.Name)
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*