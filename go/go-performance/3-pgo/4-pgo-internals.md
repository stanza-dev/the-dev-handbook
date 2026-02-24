---
source_course: "go-performance"
source_lesson: "go-performance-pgo-internals"
---

# PGO Compiler Internals

## Introduction
Understanding how the Go compiler uses PGO profiles helps you write code that benefits more from PGO and debug cases where PGO doesn't help as expected. This lesson covers the compiler's internal PGO pipeline — from profile parsing to optimization decisions.

## Key Concepts
- **Hot call graph**: The compiler builds a weighted call graph from the profile, identifying the hottest edges (caller→callee pairs).
- **Inlining budget**: Normally, Go inlines functions below ~80 nodes in the AST. PGO increases this budget for hot functions.
- **Devirtualization threshold**: The percentage of samples where a single concrete type dominates an interface call site before devirtualization kicks in.
- **Stale profile tolerance**: The compiler matches profile data to code by function name, not line number. This makes PGO robust to minor code changes.

## Real World Context
When PGO yields disappointing results, understanding internals helps diagnose why. Perhaps the hot path has no interface calls to devirtualize, functions are already small enough to inline, or the profile doesn't cover the production code path.

## Deep Dive

### Profile Parsing

The compiler reads the pprof profile and builds a weighted call graph:

```
main.handler (50 samples)
├── json.Unmarshal (30 samples)
│   ├── reflect.Value.MapIndex (20 samples)  ← hot
│   └── json.indirect (10 samples)
└── db.Query (20 samples)  ← I/O, PGO can't help
```

### Inlining Decisions

Without PGO, functions above the inlining threshold are never inlined. With PGO, hot functions get a higher budget:

```bash
# See PGO inlining decisions
go build -gcflags='-m=2' -pgo=default.pgo . 2>&1 | grep 'hot'
```

You might see output like:
```
./handler.go:42: inlining call to validate (hot)
./handler.go:55: cannot inline processOrder (cost 120, hot budget 160)
```

The "hot budget" is higher than the normal budget, allowing more functions to be inlined.

### Devirtualization in Practice

Consider an interface call:

```go
type Handler interface { Handle(r Request) Response }

func serve(h Handler, r Request) Response {
    return h.Handle(r)  // Interface dispatch
}
```

If the profile shows that `serve()` always receives `*MyHandler`, PGO rewrites this as:

```go
func serve(h Handler, r Request) Response {
    if mh, ok := h.(*MyHandler); ok {
        return mh.Handle(r)  // Direct call — faster
    }
    return h.Handle(r)  // Fallback for other types
}
```

The direct call eliminates interface dispatch overhead and may enable further inlining.

### Limitations

PGO cannot help with:
- I/O-bound code (waiting on network, disk, database)
- Code not covered by the profile
- Algorithmic complexity — PGO doesn't change your Big-O

## Common Pitfalls
1. **Expecting PGO to fix algorithmic problems** — PGO optimizes constant factors, not complexity. An O(n²) algorithm stays O(n²).
2. **Using a profile that doesn't cover the hot path** — If the production profile was collected during low-traffic hours, it may miss the critical request paths.

## Best Practices
1. **Check devirtualization with `-m=2`** — If you use interfaces heavily, verify that PGO is devirtualizing the hot call sites.
2. **Collect profiles during peak traffic** — This ensures PGO optimizes the code paths that matter most under load.

## Summary
- The compiler builds a weighted call graph from the profile to identify hot functions.
- Hot functions get increased inlining budgets, allowing larger functions to be inlined.
- Interface calls are devirtualized when one concrete type dominates in the profile.
- PGO matches by function name, not line numbers — robust to minor code changes.
- PGO cannot improve I/O-bound code or fix algorithmic complexity.

## Code Examples

**Inspecting PGO compiler decisions — see which functions are inlined and which interface calls are devirtualized**

```bash
# Check which functions PGO considers hot
go build -gcflags='-m=2' -pgo=default.pgo ./cmd/server 2>&1 | grep -i 'hot\|pgo'

# Example output:
# ./handler.go:42: PGO devirtualizing call to (*MyHandler).Handle
# ./parser.go:18: inlining call to validate (hot, cost 95, budget 160)
# ./parser.go:30: cannot inline transform (cost 200, hot budget 160)
```


## Resources

- [Profile-Guided Optimization Design](https://go.dev/doc/pgo) — Official documentation covering PGO internals and compiler behavior

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*