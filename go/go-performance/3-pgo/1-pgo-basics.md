---
source_course: "go-performance"
source_lesson: "go-performance-pgo-basics"
---

# PGO (Go 1.20+)

PGO allows the compiler to optimize code based on a runtime profile of the application. This typically results in 2-14% performance improvements.

## Workflow
1.  Collect a CPU profile from your production environment.
2.  Save it as `default.pgo` in your main package.
3.  Build: `go build` (auto-detects) or `go build -pgo=default.pgo`.

## How It Works

The compiler uses profile data to make smarter decisions:
*   **Inlining:** Inline frequently-called functions.
*   **Devirtualization:** Convert interface calls to direct calls.
*   **Register allocation:** Optimize hot paths.

## Collecting Profile

```bash
curl -o default.pgo \
  http://prod-server:6060/debug/pprof/profile?seconds=30
```

## Code Examples

**Using PGO**

```bash
# Collect from production
curl -o default.pgo http://prod-server:6060/debug/pprof/profile?seconds=30

# Build with PGO (auto-detection)
go build

# Or explicit
go build -pgo=default.pgo
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*