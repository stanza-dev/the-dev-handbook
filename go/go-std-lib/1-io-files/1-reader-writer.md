---
source_course: "go-std-lib"
source_lesson: "go-std-lib-reader-writer"
---

# The Power of Abstraction

`io.Reader` and `io.Writer` are the most important interfaces in Go. They allow you to stream data between files, network connections, buffers, and compressors without them knowing about each other.

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

## How Read Works

The `Read` method fills a provided byte slice and returns:
*   `n`: The number of bytes actually read.
*   `err`: An error if something went wrong, or `io.EOF` at end of data.

```go
buf := make([]byte, 1024)
n, err := reader.Read(buf)
if err == io.EOF {
    // End of input
}
data := buf[:n]  // Only use the bytes that were read
```

## io.Copy
You can copy from any Reader to any Writer using `io.Copy`. This streams the data, using a small constant amount of memory.

```go
// Copy file to stdout
f, _ := os.Open("file.txt")
io.Copy(os.Stdout, f)
```

## Code Examples

**Reading Data**

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


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*