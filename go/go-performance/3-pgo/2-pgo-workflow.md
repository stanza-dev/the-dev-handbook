---
source_course: "go-performance"
source_lesson: "go-performance-pgo-workflow"
---

# PGO Collection & CI Integration

## Introduction
A one-time profile is useful, but PGO shines when integrated into your CI/CD pipeline. By automatically collecting profiles and rebuilding with PGO, you ensure every release benefits from the latest production behavior. This lesson covers practical workflows for collecting, merging, and maintaining PGO profiles.

## Key Concepts
- **Profile merging**: Combining multiple pprof profiles into one using `go tool pprof -proto`. This produces a more representative profile.
- **Iterative PGO**: Each release's profile feeds the next build. Over 2-3 iterations, PGO converges to maximum effectiveness.
- **Profile representativeness**: The profile must cover your typical production workload. A profile from a quiet period won't optimize peak-load paths.

## Real World Context
At scale, teams collect profiles from multiple production instances (to avoid single-instance bias), merge them, and feed the merged profile to CI. GitHub Actions, GitLab CI, and other systems can automate this: collect weekly, commit to repo, rebuild, deploy.

## Deep Dive

### Collecting Representative Profiles

Collect from multiple instances and time periods:

```bash
# Collect from 3 production instances
curl -o prof1.pgo http://prod1:6060/debug/pprof/profile?seconds=30
curl -o prof2.pgo http://prod2:6060/debug/pprof/profile?seconds=30
curl -o prof3.pgo http://prod3:6060/debug/pprof/profile?seconds=30

# Merge into a single representative profile
go tool pprof -proto prof1.pgo prof2.pgo prof3.pgo > default.pgo
```

### CI Pipeline Integration

A typical CI workflow:

```yaml
# .github/workflows/build.yml
steps:
  - name: Build with PGO
    run: |
      # default.pgo is committed to the repo
      go build -o myapp ./cmd/myapp

  - name: Refresh profile (weekly job)
    if: github.event.schedule
    run: |
      curl -o default.pgo http://prod:6060/debug/pprof/profile?seconds=60
      git add default.pgo
      git commit -m "chore: refresh PGO profile"
```

### Iterative Improvement

PGO builds can further optimize on subsequent iterations:

1. Build v1 without PGO → deploy
2. Collect profile from v1 → build v2 with PGO → deploy
3. Collect profile from v2 → build v3 with PGO → deploy

By v3, the compiler has optimized code that was already optimized — the gains converge, typically within 2-3 iterations.

### Benchmarking PGO Impact

```bash
# Build without PGO
go build -pgo=off -o myapp-nopgo ./cmd/myapp

# Build with PGO
go build -o myapp-pgo ./cmd/myapp

# Compare with your benchmark suite
hyperfine './myapp-nopgo serve' './myapp-pgo serve'
```

## Common Pitfalls
1. **Single-instance profiles** — One instance may have unusual traffic. Always merge profiles from multiple instances.
2. **Not re-collecting after major refactors** — A profile pointing to renamed or removed functions is wasted. Refresh after major changes.

## Best Practices
1. **Commit default.pgo to your repository** — Every developer and CI build automatically uses PGO.
2. **Set up automated weekly profile refresh** — A cron-triggered CI job keeps the profile current without manual intervention.

## Summary
- Merge profiles from multiple instances for representative coverage.
- Integrate PGO into CI: commit `default.pgo`, rebuild automatically.
- PGO converges over 2-3 iterations — each release improves on the last.
- Use `-pgo=off` builds as baseline to measure PGO impact.
- Automate profile refresh with scheduled CI jobs.

## Code Examples

**Merging profiles and controlling PGO at build time — use -pgo=off to create a baseline for comparison**

```bash
# Merge profiles from multiple production instances
go tool pprof -proto prof1.pgo prof2.pgo prof3.pgo > default.pgo

# Build with explicit PGO control
go build -pgo=default.pgo -o myapp ./cmd/myapp  # Use specific profile
go build -pgo=off -o myapp ./cmd/myapp           # Disable PGO for baseline
go build -o myapp ./cmd/myapp                     # Auto-detect default.pgo
```


## Resources

- [Profile-Guided Optimization](https://go.dev/doc/pgo) — Official Go documentation covering PGO workflow and best practices

---

> 📘 *This lesson is part of the [High-Performance Go](https://stanza.dev/courses/go-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*