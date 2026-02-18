---
source_course: "go-performance"
source_lesson: "go-performance-pprof-commands"
---

# pprof Interactive Mode

```bash
go tool pprof cpu.prof
```

## Essential Commands

*   `top`: Show top functions by CPU/memory.
*   `top10`: Show top 10.
*   `top -cum`: Sort by cumulative (including callees).
*   `list funcName`: Show source code with annotations.
*   `web`: Open call graph in browser.
*   `peek funcName`: Show callers and callees.

## Filtering

*   `top -focus=regexp`: Only show matching functions.
*   `top -ignore=regexp`: Hide matching functions.

## Reading Output

```
flat  flat%   sum%  cum   cum%
10ms  50%    50%   15ms  75%   main.heavyWork
```

*   **flat**: Time spent in function itself.
*   **cum**: Time spent in function and all callees.
*   **sum%**: Cumulative percentage.

## Code Examples

**Profile Workflow**

```bash
# Common workflow
go tool pprof -http=:8080 http://localhost:6060/debug/pprof/profile

# Or save and analyze later
curl -o cpu.prof http://localhost:6060/debug/pprof/profile?seconds=30
go tool pprof cpu.prof
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*