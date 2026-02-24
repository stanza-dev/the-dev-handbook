---
source_course: "go-performance"
source_lesson: "go-performance-pgo-basics"
---

# PGO Fundamentals

## Introduction
Profile-Guided Optimization (PGO) lets the Go compiler use real-world performance data to generate faster code. Instead of making generic optimization decisions, the compiler uses a CPU profile from your production workload to optimize the exact code paths that matter most. PGO typically yields 2-14% performance improvements with zero code changes.

## Key Concepts
- **PGO (Profile-Guided Optimization)**: A compilation technique where the compiler uses runtime profiles to make better optimization decisions.
- **default.pgo**: A CPU profile file placed in your main package directory. The Go toolchain automatically detects and uses it during builds.
- **Hot path**: Code that executes frequently in production. PGO focuses optimizations on these paths.
- **Devirtualization**: Converting interface method calls to direct function calls when the profile shows only one concrete type is used.

## Real World Context
Google reported 2-14% throughput improvements across their Go services after enabling PGO. The improvement is free — no code changes, no risk of behavior differences. You collect a profile from production, add it to your repository, and rebuild. The compiler does the rest.

## Deep Dive

### How PGO Works

1. **Build your application** normally (without PGO).
2. **Deploy and collect** a CPU profile from production (30-60 seconds is sufficient).
3. **Add the profile** as `default.pgo` in the main package directory.
4. **Rebuild** — the compiler automatically uses the profile.

The compiler uses the profile to:
- **Inline hot functions** more aggressively (even past normal size limits)
- **Devirtualize interface calls** when only one concrete type is observed
- **Optimize branch prediction** by arranging code for the common path

### Placing the Profile

```
myapp/
├── main.go
├── default.pgo    ← CPU profile here
├── go.mod
└── go.sum
```

The Go toolchain detects `default.pgo` automatically — no build flags needed:

```bash
# This build uses PGO if default.pgo exists
go build -o myapp .
```

### Verifying PGO is Active

```bash
go build -v -o myapp . 2>&1 | grep -i pgo
# Should show PGO-related messages
```

You can also check with `go version -m myapp` to confirm the binary was built with PGO.

## Common Pitfalls
1. **Using benchmarks instead of production profiles** — Benchmarks test isolated functions. PGO needs a profile reflecting real production traffic patterns to optimize the right paths.
2. **Stale profiles** — If your code changes significantly, re-collect the profile. A profile from 6 months ago may optimize removed or changed code paths.
3. **Expecting huge gains on already-optimized code** — PGO helps most when there are clear hot paths with interface calls or functions just below the inlining threshold.

## Best Practices
1. **Automate profile collection** — Set up a cron job or CI step to collect fresh profiles from production weekly.
2. **Commit default.pgo to your repository** — This ensures every build benefits from PGO and makes the profile part of your version history.

## Summary
- PGO uses production CPU profiles to guide compiler optimizations.
- Place a CPU profile as `default.pgo` in your main package — the toolchain uses it automatically.
- Typical improvement: 2-14% throughput with zero code changes.
- PGO enables aggressive inlining, devirtualization, and better branch prediction.
- Refresh the profile regularly to keep it aligned with current code.

## Code Examples

**Complete PGO workflow — collect a production profile, place it as default.pgo, rebuild, and verify**

```bash
# Step 1: Collect a CPU profile from production (30 seconds)
curl -o default.pgo \
  http://prod-server:6060/debug/pprof/profile?seconds=30

# Step 2: Place it in your main package directory
mv default.pgo ./cmd/myapp/default.pgo

# Step 3: Rebuild — Go detects default.pgo automatically
go build -o myapp ./cmd/myapp

# Step 4: Verify PGO was used
go version -m myapp | grep build
# Shows: build -pgo=/path/to/default.pgo
```


## Resources

- [Profile-Guided Optimization](https://go.dev/doc/pgo) — Official Go documentation on PGO setup and usage
- [PGO Preview Blog Post](https://go.dev/blog/pgo-preview) — Go blog post explaining PGO design and real-world results

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*