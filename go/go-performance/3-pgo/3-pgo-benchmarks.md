---
source_course: "go-performance"
source_lesson: "go-performance-pgo-benchmarks"
---

# Measuring PGO Impact

## Comparing Results

```bash
# Baseline
go test -bench=. -count=10 > baseline.txt

# With PGO
go test -bench=. -count=10 -pgo=default.pgo > pgo.txt

# Compare
benchstat baseline.txt pgo.txt
```

## Understanding Results

```
name       old time/op  new time/op  delta
Process-8  100ms ± 2%   92ms ± 1%   -8.00%
```

## When PGO Helps Most

*   Hot paths that match production traffic.
*   Interface-heavy code (devirtualization).
*   Small functions called frequently (inlining).

## When It Doesn't Help

*   Code paths not in the profile.
*   Already-inlined functions.
*   I/O-bound operations.

## Code Examples

**Installing benchstat**

```bash
# Install benchstat
go install golang.org/x/perf/cmd/benchstat@latest

# Run comparison
benchstat baseline.txt pgo.txt
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*