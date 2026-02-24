---
source_course: "go-std-lib"
source_lesson: "go-std-lib-common-readers"
---

# Common Reader & Writer Types

## Introduction

The previous lesson introduced the `io.Reader` and `io.Writer` interfaces. Now it is time to meet the concrete types that implement them. The Go standard library ships dozens of Readers and Writers—from in-memory strings to network sockets to compression streams. Knowing which one to reach for in each situation is what separates a beginner from a productive Go developer.

## Key Concepts

- **strings.NewReader**: Creates a Reader from a string. Useful for tests and small in-memory data.
- **bytes.Buffer**: An in-memory buffer that implements both Reader and Writer. Great for building data incrementally.
- **os.File**: Implements Reader, Writer, and Closer. The bridge between your program and the filesystem.
- **Reader Composition**: The practice of wrapping one Reader inside another (e.g., a gzip reader wrapping a file reader) to create processing pipelines.

## Real World Context

Consider building an HTTP handler that compresses a JSON response on the fly. You create a `gzip.Writer` wrapping the `http.ResponseWriter`, then encode JSON directly into the gzip writer. The data flows from your struct through JSON encoding, through gzip compression, and out to the network—all without a single intermediate buffer holding the full response. This composition pattern is used in production systems at every scale.

## Deep Dive

### Common Readers

The standard library provides Readers for many data sources. Here is a quick reference.

```go
// Read from a string
r := strings.NewReader("Hello, Go!")

// Read from a byte slice
r := bytes.NewReader([]byte{0x48, 0x65, 0x6c, 0x6c, 0x6f})

// Read from a file
f, err := os.Open("data.txt")
defer f.Close()

// Read from an HTTP response
resp, err := http.Get("https://example.com")
defer resp.Body.Close()

// Decompress while reading
gr, err := gzip.NewReader(f)
defer gr.Close()
```

Notice the pattern: every Reader that holds resources (files, network connections, gzip state) must be closed with `defer`.

### Common Writers

Writers consume bytes and send them somewhere.

```go
// Write to standard output or standard error
os.Stdout.Write([]byte("Hello\n"))
os.Stderr.Write([]byte("Error!\n"))

// Write to a file
f, err := os.Create("output.txt")
defer f.Close()
f.Write([]byte("data"))

// Write to an in-memory buffer
var buf bytes.Buffer
buf.WriteString("Hello")
buf.WriteString(", World!")
fmt.Println(buf.String()) // "Hello, World!"

// Compress while writing
gw := gzip.NewWriter(f)
defer gw.Close()
gw.Write(data)
```

The `bytes.Buffer` is especially useful in tests where you want to capture output without touching the filesystem.

### Composing Readers

The real power of the Reader/Writer model is composition. You stack layers like building blocks. This example opens a gzipped file and decompresses it on the fly.

```go
f, _ := os.Open("data.gz")
defer f.Close()
gr, _ := gzip.NewReader(f)
defer gr.Close()
data, _ := io.ReadAll(gr)
```

The file Reader streams raw compressed bytes into the gzip Reader, which decompresses them as they arrive. `io.ReadAll` collects the decompressed output. In Go 1.26, `io.ReadAll` is 2x faster with 50% less memory allocation, making this pattern even more efficient for bounded data.

### Utility Functions

The `io` package provides several convenience functions built on top of Reader and Writer.

```go
// Read everything from a Reader into memory
data, err := io.ReadAll(reader)

// Write a string to any Writer
n, err := io.WriteString(writer, "Hello, World!")

// Copy with a custom buffer size
buf := make([]byte, 64*1024) // 64KB buffer
io.CopyBuffer(dst, src, buf)

// Limit how much you read
limited := io.LimitReader(reader, 1024*1024) // Max 1MB
```

Each of these works with any type that satisfies the interface—that is the power of Go's implicit interface implementation.

### Multi-Writer and Tee-Reader

Sometimes you need to send data to multiple destinations simultaneously.

```go
// Write to both a file and stdout at once
mw := io.MultiWriter(file, os.Stdout)
fmt.Fprintln(mw, "logged to both")

// Read from a source while also copying to a writer
tr := io.TeeReader(resp.Body, logFile)
data, _ := io.ReadAll(tr) // data goes to both 'data' and 'logFile'
```

`io.MultiWriter` fans out writes; `io.TeeReader` taps a read stream. Both are invaluable for logging and auditing.

## Common Pitfalls

1. **Forgetting to close composed Readers** — When you wrap `os.File` in `gzip.NewReader`, you must close both. The gzip reader does not close the underlying file for you. Always `defer gr.Close()` and `defer f.Close()`.
2. **Using `io.ReadAll` on unbounded network responses** — An HTTP body can be arbitrarily large. Always validate `Content-Length` or use `io.LimitReader` before calling `io.ReadAll` on untrusted input.
3. **Writing to a `bytes.Buffer` and forgetting to read it** — A Buffer grows in memory. In long-running services, if you keep writing without reading or resetting, you will leak memory.

## Best Practices

1. **Use `strings.NewReader` in tests instead of creating temp files** — It is faster, requires no cleanup, and makes tests self-contained.
2. **Use `bytes.Buffer` when you need both Reader and Writer** — It implements both interfaces, making it ideal for capturing output in tests or building data incrementally.
3. **Chain Readers for streaming pipelines** — Prefer `gzip.NewReader(file)` over reading the entire file first, then decompressing. The streaming approach uses constant memory regardless of file size.

## Summary

- The standard library provides concrete Readers (`strings.NewReader`, `bytes.NewReader`, `os.File`, `http.Response.Body`) and Writers (`os.Stdout`, `os.File`, `bytes.Buffer`, `http.ResponseWriter`).
- Readers and Writers can be composed by wrapping one inside another, creating streaming data pipelines.
- `io.ReadAll` is convenient for small data (2x faster in Go 1.26), while `io.Copy` is preferred for large or unbounded streams.
- Utilities like `io.MultiWriter`, `io.TeeReader`, and `io.LimitReader` solve common real-world patterns.
- Always close Readers and Writers that hold system resources using `defer`.

## Code Examples

**Chaining an HTTP response body through a gzip decompressor — demonstrates how Reader composition enables streaming decompression without loading the full response into memory**

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


## Resources

- [io package - Go Documentation](https://pkg.go.dev/io) — Official reference for io.ReadAll, io.Copy, and other utilities
- [strings package - Go Documentation](https://pkg.go.dev/strings) — Official reference for strings.NewReader and string utilities

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*