---
source_course: "go-std-lib"
source_lesson: "go-std-lib-json-streaming"
---

# Streaming JSON

For large files or network streams, use `json.Encoder` and `json.Decoder` instead of Marshal/Unmarshal.

## Encoder

Writes JSON directly to an io.Writer:

```go
enc := json.NewEncoder(os.Stdout)
enc.SetIndent("", "  ")  // Pretty print
enc.Encode(user)  // Writes {"name":"Alice"...}
```

## Decoder

Reads JSON from an io.Reader:

```go
dec := json.NewDecoder(resp.Body)
var result Response
if err := dec.Decode(&result); err != nil {
    log.Fatal(err)
}
```

## Processing Multiple JSON Objects

```go
dec := json.NewDecoder(file)
for dec.More() {
    var item Item
    if err := dec.Decode(&item); err != nil {
        break
    }
    process(item)
}
```

## Raw JSON

Use `json.RawMessage` to delay parsing:

```go
type Response struct {
    Type string          `json:"type"`
    Data json.RawMessage `json:"data"`  // Parse later
}
```

## Code Examples

**JSON Encoder for HTTP**

```go
func writeJSONResponse(w http.ResponseWriter, data any) {
    w.Header().Set("Content-Type", "application/json")
    enc := json.NewEncoder(w)
    if err := enc.Encode(data); err != nil {
        http.Error(w, err.Error(), 500)
    }
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*