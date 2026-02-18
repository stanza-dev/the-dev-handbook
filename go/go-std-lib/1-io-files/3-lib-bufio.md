---
source_course: "go-std-lib"
source_lesson: "go-std-lib-bufio"
---

# Why Buffer?

System calls (like reading from disk or network) are expensive. Reading byte-by-byte is slow. `bufio` wraps an `io.Reader` or `io.Writer` to buffer operations.

## bufio.Reader

```go
f, _ := os.Open("data.txt")
reader := bufio.NewReader(f)

// Read a full line efficiently
line, err := reader.ReadString('\n')

// Read until delimiter
data, err := reader.ReadBytes(':')

// Peek without consuming
bytes, err := reader.Peek(10)
```

## bufio.Writer

Collects small writes into a buffer and writes them all at once:

```go
w := bufio.NewWriter(os.Stdout)
fmt.Fprint(w, "Hello, ")
fmt.Fprint(w, "world!\n")
w.Flush() // Don't forget to flush!
```

## bufio.Scanner

For line-by-line reading:

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

## Code Examples

**Buffered Writer**

```go
func main() {
    w := bufio.NewWriter(os.Stdout)
    fmt.Fprint(w, "Hello, ")
    fmt.Fprint(w, "world!\n")
    w.Flush() // Don't forget to flush!
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*