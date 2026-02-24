---
source_course: "go-std-lib"
source_lesson: "go-std-lib-json-streaming"
---

# Streaming JSON

## Introduction

The previous lesson covered `json.Marshal` and `json.Unmarshal`, which work with complete byte slices in memory. But what happens when you need to encode JSON directly into an HTTP response or decode a stream of JSON objects from a large file? That is where `json.Encoder` and `json.Decoder` come in. They work with `io.Reader` and `io.Writer`, enabling streaming JSON without buffering the entire payload.

## Key Concepts

- **json.Encoder**: Wraps an `io.Writer` and writes JSON directly to it. No intermediate `[]byte` allocation.
- **json.Decoder**: Wraps an `io.Reader` and reads JSON objects one at a time from the stream.
- **json.RawMessage**: A raw, unparsed JSON value. Useful when you need to delay parsing or forward JSON without decoding it.
- **dec.More()**: Returns true if there are more elements in the current JSON array or object, enabling iteration over a stream of values.

## Real World Context

Imagine you are building a microservice that receives a newline-delimited JSON (NDJSON) stream from a data pipeline—each line is a separate JSON object. Using `json.Decoder`, you can parse each object as it arrives without waiting for the entire stream to finish. On the response side, `json.Encoder` writes JSON directly to the `http.ResponseWriter`, avoiding the memory cost of marshalling to a byte slice first. For high-throughput services processing thousands of requests per second, this difference matters.

## Deep Dive

### json.Encoder

An Encoder writes JSON directly to any `io.Writer`. This is the standard way to send JSON responses in HTTP handlers.

```go
enc := json.NewEncoder(os.Stdout)
enc.SetIndent("", "  ")  // Pretty print
enc.Encode(user)          // Writes {"name":"Alice"...}\n
```

Note that `Encode` appends a trailing newline after the JSON. This is intentional and follows the NDJSON convention, but it means the output differs slightly from `json.Marshal`.

For HTTP handlers, Encoder avoids allocating an intermediate byte slice.

```go
func writeJSON(w http.ResponseWriter, data any) {
    w.Header().Set("Content-Type", "application/json")
    enc := json.NewEncoder(w)
    if err := enc.Encode(data); err != nil {
        http.Error(w, err.Error(), 500)
    }
}
```

This streams the JSON directly to the network socket through the ResponseWriter.

### json.Decoder

A Decoder reads JSON from any `io.Reader`. It is the streaming counterpart to `json.Unmarshal`.

```go
dec := json.NewDecoder(resp.Body)
var result Response
if err := dec.Decode(&result); err != nil {
    log.Fatal(err)
}
```

The Decoder reads only enough bytes to parse one complete JSON value. The rest of the stream remains unconsumed in the Reader, ready for the next `Decode` call.

### Processing Multiple JSON Objects

When a stream contains multiple JSON objects (like an NDJSON file or a JSON array), use `dec.More()` to iterate.

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

`dec.More()` returns `true` if there are more values to decode in the current array or top-level stream. This is memory-efficient because only one `Item` is in memory at a time, regardless of how many objects are in the file.

### Decoding JSON Arrays

For a proper JSON array (`[{...}, {...}]`), you need to consume the opening bracket first.

```go
dec := json.NewDecoder(file)
// Read opening bracket
dec.Token() // returns json.Delim('[')
for dec.More() {
    var item Item
    dec.Decode(&item)
    process(item)
}
// Read closing bracket
dec.Token() // returns json.Delim(']')
```

The `Token()` method reads the next JSON token (delimiters, strings, numbers) without decoding into a struct.

### json.RawMessage

Use `json.RawMessage` when you want to delay parsing part of a JSON document. The raw bytes are preserved as-is.

```go
type Envelope struct {
    Type string          `json:"type"`
    Data json.RawMessage `json:"data"`  // Parse later based on Type
}

var env Envelope
json.Unmarshal(payload, &env)

switch env.Type {
case "user":
    var u User
    json.Unmarshal(env.Data, &u)
case "order":
    var o Order
    json.Unmarshal(env.Data, &o)
}
```

This pattern is common in event-driven systems where the outer envelope tells you what type of payload is inside, and you defer full parsing until you know the type.

## Common Pitfalls

1. **Forgetting the trailing newline from Encoder** — `enc.Encode()` appends `\n` after the JSON. If your protocol does not expect it, use `json.Marshal` and `w.Write()` instead.
2. **Using Decoder when you have a complete byte slice** — If you already have all the JSON in memory as `[]byte`, `json.Unmarshal` is simpler and avoids the overhead of wrapping in a Reader. Use Decoder for streams, Unmarshal for byte slices.
3. **Not checking `dec.More()` in array iteration** — Calling `Decode` without checking `More()` reads past the end of a JSON array and returns an error. Always use the `More()` loop pattern.

## Best Practices

1. **Use Encoder/Decoder for HTTP handlers** — They avoid intermediate allocations and stream data directly to/from the network, reducing memory usage under load.
2. **Use `json.RawMessage` for polymorphic JSON** — When a field can contain different structures based on a type discriminator, parse the envelope first and defer the inner payload.
3. **Use `DisallowUnknownFields()` for strict parsing** — Call `dec.DisallowUnknownFields()` before `Decode` to reject JSON with unexpected keys, catching typos and API mismatches early.

## Summary

- `json.Encoder` writes JSON directly to an `io.Writer`, avoiding intermediate byte slice allocations.
- `json.Decoder` reads JSON from an `io.Reader` one value at a time, enabling memory-efficient stream processing.
- Use `dec.More()` to iterate over multiple JSON objects or array elements in a stream.
- `json.RawMessage` preserves raw JSON bytes for deferred or conditional parsing.
- Prefer Encoder/Decoder for I/O streams and Marshal/Unmarshal for in-memory byte slices.

## Code Examples

**Streaming JSON directly to an HTTP response using json.NewEncoder — avoids allocating the entire JSON byte slice in memory**

```go
func writeJSONResponse(w http.ResponseWriter, data any) {
    w.Header().Set("Content-Type", "application/json")
    enc := json.NewEncoder(w)
    if err := enc.Encode(data); err != nil {
        http.Error(w, err.Error(), 500)
    }
}
```


## Resources

- [encoding/json package - Go Documentation](https://pkg.go.dev/encoding/json) — Official reference for json.Encoder, json.Decoder, and streaming JSON
- [JSON and Go - Go Blog](https://go.dev/blog/json) — Official blog post on working with JSON in Go

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*