---
source_course: "go-std-lib"
source_lesson: "go-std-lib-os-files"
---

# The os Package

The `os` package provides platform-independent file operations.

## Opening Files

```go
// Read-only
f, err := os.Open("file.txt")
defer f.Close()

// Create or truncate
f, err := os.Create("output.txt")

// Full control
f, err := os.OpenFile("log.txt",
    os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
```

## Reading and Writing

```go
// Read entire file (convenience function)
data, err := os.ReadFile("config.json")

// Write entire file
err := os.WriteFile("output.txt", data, 0644)
```

## Directory Operations

```go
// List directory
entries, err := os.ReadDir(".")
for _, e := range entries {
    fmt.Println(e.Name(), e.IsDir())
}

// Create directory
err := os.Mkdir("newdir", 0755)
err := os.MkdirAll("path/to/deep/dir", 0755)

// Remove
err := os.Remove("file.txt")
err := os.RemoveAll("dir")  // Recursive
```

## File Info

```go
info, err := os.Stat("file.txt")
fmt.Println(info.Size())    // Size in bytes
fmt.Println(info.ModTime()) // Modification time
fmt.Println(info.IsDir())   // Is directory?
```

## Code Examples

**Copy File**

```go
func copyFile(src, dst string) error {
    in, err := os.Open(src)
    if err != nil {
        return err
    }
    defer in.Close()
    
    out, err := os.Create(dst)
    if err != nil {
        return err
    }
    defer out.Close()
    
    _, err = io.Copy(out, in)
    return err
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*