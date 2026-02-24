---
source_course: "go-std-lib"
source_lesson: "go-std-lib-reader-writer"
---

# The Reader & Writer Interfaces

## Introduction

If you learn only two interfaces in Go, make them `io.Reader` and `io.Writer`. These tiny contracts power everything from file operations to HTTP servers to compression pipelines. Once you understand how they work, you can plug any data source into any data sink without either side knowing about the other.

## Key Concepts

- **io.Reader**: An interface with a single `Read(p []byte) (n int, err error)` method. Any type that can produce bytes implements it.
- **io.Writer**: An interface with a single `Write(p []byte) (n int, err error)` method. Any type that can consume bytes implements it.
- **io.EOF**: A sentinel error value that signals the end of a data stream. It is not a failure—it is the normal way a Reader says "I have no more data."
- **io.Copy**: A utility that streams bytes from any Reader to any Writer using a small internal buffer.

## Real World Context

Imagine you are building a service that downloads a gzipped CSV file from S3, decompresses it, and writes the results into a database. With Reader and Writer, you chain an HTTP response body into a gzip decompressor into a CSV parser—all streaming, all using constant memory. Without these interfaces, you would need to load the entire file into memory first, which could crash your service on large datasets.

## Deep Dive

### The Interface Definitions

The `io` package defines the two core interfaces that nearly every I/O operation in Go relies on.

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

Notice how minimal they are: one method each. This simplicity is intentional—it makes them easy to implement and easy to compose.

### How Read Works

The `Read` method fills a caller-provided byte slice and returns two values: `n` (the number of bytes actually placed into the slice) and `err` (an error, or `io.EOF` when data is exhausted). You must always process the `n` bytes before checking `err`, because a Read can return both data and `io.EOF` in the same call.

```go
buf := make([]byte, 1024)
n, err := reader.Read(buf)
data := buf[:n]  // Only use the bytes that were actually read
if err == io.EOF {
    // End of input reached
}
```

The key detail here is `buf[:n]`. The buffer may be 1024 bytes, but the Reader might only fill 37 of them. Always slice down to `n`.

### Reading in a Loop

In practice you rarely call Read once. You loop until `io.EOF`.

```go
var all []byte
buf := make([]byte, 512)
for {
    n, err := reader.Read(buf)
    all = append(all, buf[:n]...)
    if err == io.EOF {
        break
    }
    if err != nil {
        log.Fatal(err)
    }
}
```

This loop processes bytes returned on every call, even the final one that also returns `io.EOF`. That ordering matters.

### Streaming with io.Copy

You can copy from any Reader to any Writer using `io.Copy`. It uses a small 32KB internal buffer, so even multi-gigabyte transfers consume only kilobytes of memory.

```go
f, _ := os.Open("file.txt")
defer f.Close()
io.Copy(os.Stdout, f)
```

This streams the file contents directly to standard output. No intermediate byte slice holds the entire file.

### io.ReadAll

`io.ReadAll` reads every byte from a Reader into memory. In Go 1.26, `io.ReadAll` is 2x faster with 50% less memory allocation than in previous versions, but it still loads the entire content into RAM—so use it only when you know the data fits comfortably in memory.

```go
data, err := io.ReadAll(reader)
if err != nil {
    log.Fatal(err)
}
```

This is convenient for small payloads like config files or API responses, but dangerous for unbounded streams.

## Common Pitfalls

1. **Ignoring `n` before checking `err`** — A Read call can return both valid bytes and `io.EOF` simultaneously. If you check `err` first and break, you lose those final bytes. Always process `buf[:n]` before inspecting the error.
2. **Using the full buffer instead of `buf[:n]`** — The slice you pass to `Read` is not guaranteed to be filled. Using `buf` instead of `buf[:n]` includes leftover garbage from previous reads.
3. **Calling `io.ReadAll` on unbounded streams** — Reading an entire HTTP body or a multi-GB file into memory can exhaust RAM and crash your program. Use `io.Copy` or buffered reading for large or unknown-size data.

## Best Practices

1. **Prefer `io.Copy` for transferring data between a Reader and Writer** — It handles the read-write loop, buffer management, and EOF detection for you with constant memory usage.
2. **Always `defer Close()` on Readers and Writers that implement `io.Closer`** — Files, HTTP bodies, and gzip readers all hold OS resources. Forgetting to close them causes resource leaks.
3. **Use `io.LimitReader` to cap how much data you read** — When reading from untrusted sources (e.g., user uploads), wrap the Reader with `io.LimitReader(r, maxBytes)` to prevent memory exhaustion.

## Summary

- `io.Reader` and `io.Writer` are single-method interfaces that form the backbone of Go's I/O system.
- `Read` fills a byte slice and returns `n` bytes read plus an error; always process `buf[:n]` before checking `err`.
- `io.Copy` streams data between any Reader and Writer using constant memory (32KB buffer).
- `io.ReadAll` reads everything into memory (2x faster in Go 1.26) but should only be used for small, bounded data.
- These interfaces enable powerful composition: you can chain files, network connections, compressors, and buffers without any of them knowing about each other.

## Code Examples

**Reading from a strings.Reader in 8-byte chunks — notice how n tracks bytes actually read and err signals EOF when complete**

```go
func main() {
    r := strings.NewReader("Hello, Reader!")
    b := make([]byte, 8)
    for {
        n, err := r.Read(b)
        fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
        if err == io.EOF {
            break
        }
    }
}
```


## Resources

- [io package - Go Documentation](https://pkg.go.dev/io) — Official Go documentation for the io package, including Reader and Writer interfaces

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*