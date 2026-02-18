---
source_course: "go-performance"
source_lesson: "go-performance-pgo-workflow"
---

# PGO Workflow

## 1. Baseline Build

First, build and deploy without PGO to collect real traffic patterns.

## 2. Collect Representative Profile

*   Profile during typical production load.
*   Collect for at least 30 seconds.
*   Profile multiple times if load varies.

## 3. Merge Profiles (Optional)

```bash
go tool pprof -proto profile1.pprof profile2.pprof > merged.pgo
```

## 4. Build with PGO

```bash
cp merged.pgo default.pgo
go build
```

## 5. Iterate

*   Update profiles periodically.
*   Major code changes may need new profiles.
*   Stale profiles can still help but fresh is better.

## Version Control

Commit `default.pgo` to your repository so CI/CD uses it.

## Code Examples

**PGO CI/CD**

```bash
# CI/CD example
# 1. Build with existing profile
go build -pgo=default.pgo -o myapp

# 2. Deploy and collect new profile
# 3. Replace default.pgo
# 4. Commit updated profile
```


---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*