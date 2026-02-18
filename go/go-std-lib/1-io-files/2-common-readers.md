---
source_course: "go-std-lib"
source_lesson: "go-std-lib-common-readers"
---

# Common Implementations

The standard library provides many Reader and Writer implementations.

## Readers

*   `strings.NewReader(s)`: Read from a string.
*   `bytes.NewReader(b)`: Read from a byte slice.
*   `os.File`: Read from files.
*   `http.Response.Body`: Read from HTTP responses.
*   `compress/gzip.Reader`: Decompress while reading.

## Writers

*   `os.Stdout`, `os.Stderr`: Write to console.
*   `os.File`: Write to files.
*   `bytes.Buffer`: Write to an in-memory buffer.
*   `http.ResponseWriter`: Write HTTP responses.
*   `compress/gzip.Writer`: Compress while writing.

## Composing Readers/Writers

You can chain them together:

```go
// Read gzipped file, decompress on the fly
f, _ := os.Open("data.gz")
gr, _ := gzip.NewReader(f)
data, _ := io.ReadAll(gr)
```

## io.ReadAll and io.WriteString

```go
// Read entire content (use carefully with large files)
data, err := io.ReadAll(reader)

// Write a string
io.WriteString(writer, "Hello, World!")
```

## Code Examples

**Chained Readers**

```go
func downloadAndDecompress(url string) ([]byte, error) {
    resp, err := http.Get(url)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    gr, err := gzip.NewReader(resp.Body)
    if err != nil {
        return nil, err
    }
    defer gr.Close()
    
    return io.ReadAll(gr)
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*