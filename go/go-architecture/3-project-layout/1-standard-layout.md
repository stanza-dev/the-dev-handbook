---
source_course: "go-architecture"
source_lesson: "go-architecture-standard-layout"
---

# Standard Go Project Layout

## Introduction
Go does not enforce a project directory structure, but the community follows well-established conventions. Following these conventions makes your project immediately navigable to other Go developers. Understanding the standard layout is essential for building maintainable applications.

## Key Concepts
- **/cmd**: Contains main packages—one subdirectory per binary your project produces.
- **/internal**: Private code that the Go compiler prevents other modules from importing.
- **/pkg**: Library code that external projects may import (use sparingly).
- **/api**: API definitions such as OpenAPI specs, gRPC protobuf files, or JSON schemas.

## Real World Context
You join a new Go team and open the repository. If it follows the standard layout, you immediately know where to find the entry points (/cmd), the business logic (/internal), and the API contracts (/api). Without conventions, every project is a puzzle.

## Deep Dive
A typical Go project structure looks like this:

```
my-project/
  cmd/
    api-server/
      main.go
    cli/
      main.go
  internal/
    auth/
    database/
    handlers/
  pkg/
    publiclib/
  api/
    openapi.yaml
  go.mod
```

The `/cmd` directory holds main packages. Each subdirectory produces one binary. Keep these files minimal—they should parse config, wire dependencies, and call into `/internal`.

The `/internal` directory is special. The Go compiler enforces that packages under `/internal` can only be imported by code in the parent directory tree. For example, `myproject/internal/auth` can be imported by `myproject/cmd/server` but not by `otherproject/...`.

The `/pkg` directory is for library code that is safe for external consumption. Modern Go projects often skip `/pkg` entirely and put public packages at the root.

## Common Pitfalls
1. **Generic package names like `utils` or `helpers`** — These become dumping grounds. Name packages by what they do: `auth`, `database`, `email`.
2. **Cyclic dependencies** — Package A imports B and B imports A. This is a compile error in Go. Use interfaces or a shared types package to break cycles.

## Best Practices
1. **Keep /cmd packages thin** — They should only wire dependencies and start the application. Business logic belongs in /internal.
2. **Use /internal liberally** — Default to private. Only expose packages in /pkg when you have external consumers.

## Summary
- Follow the community-standard layout: /cmd, /internal, /pkg, /api.
- The Go compiler enforces /internal visibility—external modules cannot import it.
- Avoid generic package names and cyclic dependencies.
- Keep main packages thin; put logic in /internal.

## Code Examples

**Standard Go project layout showing the four conventional top-level directories and their purposes**

```bash
my-project/
  cmd/
    api-server/
      main.go          # Entry point for the API server binary
    cli/
      main.go          # Entry point for the CLI tool binary
  internal/
    auth/              # Authentication logic (private)
    database/          # Database access layer (private)
    handlers/          # HTTP handlers (private)
  pkg/
    publiclib/         # Code safe for external import
  api/
    openapi.yaml       # API specification
  go.mod
```


## Resources

- [Go Project Layout](https://github.com/golang-standards/project-layout) — Community-maintained reference for standard Go project structure
- [Effective Go](https://go.dev/doc/effective_go) — Official guide covering Go conventions including package naming

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*