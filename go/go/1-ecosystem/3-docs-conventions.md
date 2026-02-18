---
source_course: "go"
source_lesson: "go-docs-conventions"
---

# Documentation & Go Doc

## Introduction

Great code tells you *what* happens; great documentation tells you *why*. Go elevates documentation from an afterthought to a first-class language feature. Comments written in a specific format are automatically transformed into browsable documentation on pkg.go.dev and accessible via the `go doc` command.

## Key Concepts

**Doc Comments**: Comments placed directly before a declaration that become part of the generated documentation.

**go doc**: Command-line tool for reading package documentation locally.

**pkg.go.dev**: The official website hosting documentation for all public Go packages.

**Example Functions**: Executable documentation that appears in docs and runs as tests.

## Real World Context

When you import a package, you rely on its documentation to understand the API. Well-documented packages get adopted; poorly documented ones get abandoned. At companies like Google, documentation is reviewed as carefully as code. The Go team maintains strict documentation standards for the standard library, setting expectations for the ecosystem.

## Deep Dive

### Doc Comment Format

Start comments with the identifier name:

```go
// Sum returns the sum of a and b.
// It handles integer overflow by wrapping.
func Sum(a, b int) int {
    return a + b
}
```

### Package Documentation

Document the entire package with a comment before the `package` clause:

```go
// Package auth provides authentication and authorization
// utilities for web applications.
//
// It supports JWT tokens, OAuth 2.0, and session-based auth.
package auth
```

### Using go doc

```bash
go doc fmt           # Package overview
go doc fmt.Println   # Specific function
go doc -all fmt      # All exported symbols
go doc -src fmt.Scan # Show source code
```

### Example Functions

```go
func ExampleSum() {
    result := Sum(2, 3)
    fmt.Println(result)
    // Output: 5
}
```

These appear in documentation and are verified by `go test`.

## Common Pitfalls

1. **Not starting with the identifier name**: `// This function sums...` is wrong. Write `// Sum returns...` instead.

2. **Documenting unexported functions**: While you can, they won't appear in `go doc` output or pkg.go.dev. Focus documentation effort on exported APIs.

3. **Forgetting the Output comment**: Example functions without `// Output:` comments don't verify their output during tests.

## Best Practices

- Document every exported type, function, and constant.
- Use complete sentences with proper punctuation.
- Include code examples for complex APIs using `Example` functions.
- Link to related functions: `// See also: [OtherFunc]`.
- Add a `doc.go` file for extensive package documentation.

## Summary

Go documentation is extracted from comments preceding declarations. Doc comments must start with the identifier name and use complete sentences. The `go doc` command provides local access, while pkg.go.dev hosts public documentation. Example functions serve as both documentation and executable tests.

## Code Examples

**Documenting a Struct**

```go
// User represents a registered user in the system.
type User struct {
    Name string
    // Email is the unique identifier.
    Email string
}
```


## Resources

- [Go Doc Comments](https://go.dev/doc/comment) — Official guide to writing doc comments
- [pkg.go.dev About](https://pkg.go.dev/about) — Information about the Go package documentation site

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*