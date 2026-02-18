---
source_course: "go-performance"
source_lesson: "go-performance-pprof-tool"
---

# pprof

Go has a built-in profiler. It can profile CPU, Memory (Heap), Block (synchronization), and Goroutines.

## Collecting a Profile

```go
import _ "net/http/pprof"
// ...
go http.ListenAndServe(":6060", nil)
```

Then run:
```bash
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
```

## Profile Types

*   `/debug/pprof/profile`: CPU profile.
*   `/debug/pprof/heap`: Memory allocations.
*   `/debug/pprof/goroutine`: Goroutine stacks.
*   `/debug/pprof/block`: Blocking profile.
*   `/debug/pprof/mutex`: Contention profile.

## Reading Profiles
Use `top` to see the most expensive functions, or `web` to visualize the call graph.

See [Profiling Go Programs](https://go.dev/blog/pprof).

## Code Examples

**Visualizing Profile**

```bash
go tool pprof -http=:8080 cpu.prof
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*