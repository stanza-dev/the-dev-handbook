---
source_course: "go-std-lib"
source_lesson: "go-std-lib-os-files"
---

# File Operations with os Package

## Introduction

At some point, every program needs to read a config file, write a log, or list a directory. The `os` package is Go's platform-independent gateway to the filesystem. It gives you everything from simple one-line file reads to fine-grained control over file permissions and flags. If you have used C's `fopen` or Python's `open`, Go's approach will feel familiar but safer thanks to explicit error handling.

## Key Concepts

- **os.Open / os.Create**: Convenience functions for the two most common file operations—reading and writing.
- **os.OpenFile**: The full-control variant that accepts flags (`O_APPEND`, `O_CREATE`, etc.) and permission bits.
- **os.ReadFile / os.WriteFile**: One-liner convenience functions that read or write an entire file in a single call.
- **os.FileInfo**: Metadata about a file—size, modification time, permissions, and whether it is a directory.

## Real World Context

Consider a CLI tool that processes configuration. It needs to check if a config file exists, read it, apply defaults for missing fields, and write the merged result back. Along the way it may need to create directories, set permissions, and handle the case where the file is missing versus corrupt. The `os` package handles all of these operations portably across Linux, macOS, and Windows.

## Deep Dive

### Opening Files

Go provides three ways to open files, each for a different level of control.

```go
// Read-only (most common for reading)
f, err := os.Open("file.txt")
if err != nil {
    log.Fatal(err)
}
defer f.Close()

// Create or truncate (for writing new content)
f, err := os.Create("output.txt")
if err != nil {
    log.Fatal(err)
}
defer f.Close()

// Full control with flags and permissions
f, err := os.OpenFile("log.txt",
    os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
if err != nil {
    log.Fatal(err)
}
defer f.Close()
```

`os.Open` is shorthand for `os.OpenFile(name, os.O_RDONLY, 0)`. `os.Create` is shorthand for `os.OpenFile(name, os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0666)`. When you need append mode or custom permissions, use `os.OpenFile` directly.

### Reading and Writing Entire Files

For small files where you want the entire content in memory, Go provides one-liner convenience functions.

```go
// Read entire file into a byte slice
data, err := os.ReadFile("config.json")
if err != nil {
    log.Fatal(err)
}

// Write entire byte slice to a file
err = os.WriteFile("output.txt", data, 0644)
if err != nil {
    log.Fatal(err)
}
```

`os.ReadFile` opens the file, reads everything, and closes it. `os.WriteFile` creates or truncates the file, writes the data, and closes it. Both handle cleanup automatically. For large files, prefer streaming with `io.Copy` instead.

### Directory Operations

The `os` package provides functions for creating, listing, and removing directories.

```go
// List directory entries
entries, err := os.ReadDir(".")
for _, e := range entries {
    fmt.Println(e.Name(), e.IsDir())
}

// Create a single directory
err := os.Mkdir("newdir", 0755)

// Create nested directories (like mkdir -p)
err := os.MkdirAll("path/to/deep/dir", 0755)

// Remove a single file or empty directory
err := os.Remove("file.txt")

// Remove recursively (like rm -rf)
err := os.RemoveAll("dir")
```

`os.ReadDir` returns `[]os.DirEntry`, which is more efficient than the older `ioutil.ReadDir` because it does not call `Stat` on every entry. Use `MkdirAll` over `Mkdir` when you are not sure whether parent directories exist.

### File Info and Stat

You can inspect file metadata without opening the file using `os.Stat`.

```go
info, err := os.Stat("file.txt")
if err != nil {
    if os.IsNotExist(err) {
        fmt.Println("File does not exist")
    }
    log.Fatal(err)
}
fmt.Println(info.Size())    // Size in bytes
fmt.Println(info.ModTime()) // Modification time
fmt.Println(info.IsDir())   // Is directory?
fmt.Println(info.Mode())    // Permission bits
```

`os.IsNotExist(err)` is the idiomatic way to check if a file is missing. In Go 1.26, you can also use `errors.Is(err, fs.ErrNotExist)` for the same check.

### Temporary Files and Directories

For tests and intermediate processing, you can create temporary files and directories.

```go
// Create a temp file in the default temp directory
f, err := os.CreateTemp("", "prefix-*.txt")
defer os.Remove(f.Name())
defer f.Close()

// Create a temp directory
dir, err := os.MkdirTemp("", "workdir-*")
defer os.RemoveAll(dir)
```

The `*` in the pattern is replaced with a random string. These functions are essential for writing tests that do not pollute the filesystem.

## Common Pitfalls

1. **Forgetting `defer f.Close()`** — Every opened file holds a file descriptor. In a loop or long-running service, leaked file descriptors accumulate until the OS limit is hit and all opens fail.
2. **Using `os.Create` when you want to append** — `os.Create` truncates the file to zero length. To append, use `os.OpenFile` with `os.O_APPEND|os.O_CREATE|os.O_WRONLY`.
3. **Checking file existence with `os.Stat` before opening** — This is a TOCTOU (time-of-check-time-of-use) race. Another process could delete or create the file between your `Stat` and `Open`. Instead, just open the file and handle the error.

## Best Practices

1. **Use `os.ReadFile` / `os.WriteFile` for small config files and data** — They are concise, handle cleanup, and reduce the chance of forgetting to close.
2. **Use `os.MkdirAll` instead of `os.Mkdir`** — It is idempotent: if the directory already exists, it does nothing. This avoids errors in scripts that run multiple times.
3. **Use `os.CreateTemp` in tests** — Temporary files with cleanup prevent test pollution and make tests safe to run in parallel.

## Summary

- `os.Open`, `os.Create`, and `os.OpenFile` provide increasing levels of control for file access.
- `os.ReadFile` and `os.WriteFile` are one-liner convenience functions for small files.
- `os.ReadDir`, `os.Mkdir`, `os.MkdirAll`, `os.Remove`, and `os.RemoveAll` cover directory operations.
- `os.Stat` retrieves file metadata; use `os.IsNotExist` to check for missing files.
- Always `defer f.Close()` after opening a file and prefer `os.CreateTemp` for temporary files in tests.

## Code Examples

**Copying a file using os.Open, os.Create, and io.Copy — the deferred Close calls ensure files are properly closed even if an error occurs**

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


## Resources

- [os package - Go Documentation](https://pkg.go.dev/os) — Official Go documentation for file and directory operations

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*