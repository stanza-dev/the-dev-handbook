---
source_course: "go-architecture"
source_lesson: "go-architecture-standard-layout"
---

# Project Structure

While not enforced by the compiler, the community follows a standard layout:

*   `/cmd`: Main applications. One directory per binary (e.g., `/cmd/server`, `/cmd/cli`).
*   `/internal`: Private application and library code. The Go compiler enforces that packages here cannot be imported by other modules.
*   `/pkg`: Library code that is ok to use by external applications (use sparingly).
*   `/api`: OpenAPI/gRPC specifications.

## Package Design

*   Avoid generic names like `utils` or `helpers`.
*   Avoid cyclic dependencies (Package A imports B, B imports A). This is a compile error in Go.
*   Keep packages focused on a single concern.

## Code Examples

**Directory Structure**

```bash
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
  go.mod
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*