---
source_course: "go-std-lib"
source_lesson: "go-std-lib-bufio"
---

# Buffered I/O

## Introduction

Every call to `Read` or `Write` on an `os.File` triggers a system call, and system calls are expensive. If you read a file one byte at a time, you make one system call per byte—thousands of kernel transitions for a small file. The `bufio` package solves this by wrapping any `io.Reader` or `io.Writer` with an in-memory buffer, batching many small operations into fewer large ones.

## Key Concepts

- **bufio.Reader**: Wraps an `io.Reader` with a read-ahead buffer (default 4KB). Provides convenience methods like `ReadString`, `ReadBytes`, and `Peek`.
- **bufio.Writer**: Wraps an `io.Writer` with a write buffer. Collects small writes and flushes them in larger batches.
- **bufio.Scanner**: A high-level abstraction for reading delimited data (lines by default) from any Reader.
- **Flush**: The act of writing all buffered data to the underlying Writer. Forgetting to flush means losing data.

## Real World Context

Imagine you are writing a log file rotator that reads an access log line by line, filters for error entries, and writes them to a separate file. Without buffering, each `ReadString` would trigger a system call for each line, and each `WriteString` another. With `bufio`, the reads pull 4KB chunks from disk at a time, and the writes batch into a single system call. For a million-line log file, this can reduce system calls from 2 million to a few hundred.

## Deep Dive

### bufio.Reader

You create a buffered reader by wrapping any `io.Reader`. The buffer pre-fetches data so subsequent reads can be served from memory.

```go
f, _ := os.Open("data.txt")
reader := bufio.NewReader(f)

// Read a full line (up to and including the delimiter)
line, err := reader.ReadString('\n')

// Read until a specific byte delimiter
data, err := reader.ReadBytes(':')

// Peek at the next 10 bytes without consuming them
peeked, err := reader.Peek(10)
```

`ReadString` and `ReadBytes` return data up to and including the delimiter. `Peek` lets you look ahead without advancing the read position—useful for parsing protocols where you need to inspect a header before deciding how to proceed.

You can also control the buffer size when the default 4KB is not enough.

```go
// Create a reader with a 64KB buffer
reader := bufio.NewReaderSize(f, 64*1024)
```

This is helpful when reading files with very long lines or when you want to minimize system calls even further.

### bufio.Writer

A buffered writer collects small writes into a buffer and sends them to the underlying Writer in one batch when the buffer is full or when you explicitly flush.

```go
w := bufio.NewWriter(os.Stdout)
fmt.Fprint(w, "Hello, ")
fmt.Fprint(w, "world!\n")
w.Flush() // Don't forget to flush!
```

Without the `Flush()` call, the text stays in the buffer and never reaches `os.Stdout`. This is the single most common mistake with buffered writers.

### bufio.Scanner

For the common case of reading line by line, `bufio.Scanner` provides a cleaner API than `ReadString`.

```go
scanner := bufio.NewScanner(file)
for scanner.Scan() {
    line := scanner.Text()
    fmt.Println(line)
}
if err := scanner.Err(); err != nil {
    log.Fatal(err)
}
```

`Scan()` returns `true` as long as there is another token (line by default). `Text()` returns the current token as a string without the trailing delimiter. Always check `scanner.Err()` after the loop to catch I/O errors distinct from EOF.

### Custom Split Functions

By default, Scanner splits on newlines, but you can provide a custom split function for other delimiters.

```go
scanner := bufio.NewScanner(strings.NewReader("hello world foo"))
scanner.Split(bufio.ScanWords) // Split on whitespace instead
for scanner.Scan() {
    fmt.Println(scanner.Text()) // prints: hello, world, foo
}
```

Built-in split functions include `bufio.ScanLines` (default), `bufio.ScanWords`, `bufio.ScanRunes`, and `bufio.ScanBytes`.

### Scanner Buffer Limits

By default, Scanner has a maximum token size of 64KB. If a single line exceeds this, `Scan()` returns `false` and `Err()` returns `bufio.ErrTooLong`. You can increase the limit.

```go
scanner := bufio.NewScanner(file)
buf := make([]byte, 0, 1024*1024) // 1MB max
scanner.Buffer(buf, 1024*1024)
```

This sets the maximum token size to 1MB, which is necessary when processing files with very long lines like base64-encoded data or minified JSON.

## Common Pitfalls

1. **Forgetting to call `Flush()` on a `bufio.Writer`** — Data remains in the buffer and never reaches the destination. Always use `defer w.Flush()` immediately after creating the writer.
2. **Ignoring `scanner.Err()` after a scan loop** — If the loop ends due to an I/O error (not EOF), `scanner.Err()` holds the error. Skipping this check silently drops errors.
3. **Hitting the default Scanner buffer limit** — Lines longer than 64KB cause `Scan()` to fail silently. If you process files with long lines, call `scanner.Buffer()` to increase the limit.

## Best Practices

1. **Use `bufio.Scanner` for line-by-line reading** — It is cleaner than `ReadString('\n')` and handles edge cases like missing trailing newlines.
2. **Wrap network connections with `bufio.Reader`** — Network reads are especially expensive in system calls. Buffering TCP reads dramatically improves throughput for protocol parsing.
3. **Always `defer Flush()` immediately after creating a `bufio.Writer`** — This ensures data is written even if you return early due to an error.

## Summary

- `bufio` reduces system calls by batching small reads and writes into larger operations through an in-memory buffer.
- `bufio.Reader` provides `ReadString`, `ReadBytes`, and `Peek` for flexible buffered reading.
- `bufio.Writer` collects writes and requires an explicit `Flush()` to send data to the underlying Writer.
- `bufio.Scanner` is the idiomatic way to read delimited data (lines, words, runes) with customizable split functions.
- The default Scanner token limit is 64KB; increase it with `scanner.Buffer()` for files with long lines.

## Code Examples

**Using a buffered writer to batch small writes — Flush() must be called to ensure all buffered data reaches os.Stdout**

```go
func main() {
    w := bufio.NewWriter(os.Stdout)
    fmt.Fprint(w, "Hello, ")
    fmt.Fprint(w, "world!\n")
    w.Flush() // Don't forget to flush!
}
```


## Resources

- [bufio package - Go Documentation](https://pkg.go.dev/bufio) — Official Go documentation for buffered I/O operations

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*