---
source_course: "typescript-production"
source_lesson: "typescript-production-compilation-performance"
---

# Compilation Performance

## Introduction
As TypeScript projects grow, compilation can take minutes. Understanding what slows the compiler down and how to optimize it saves developer time every day. TypeScript provides built-in tools for profiling, incremental builds, and targeted optimizations.

## Key Concepts
- **Incremental Compilation**: Reusing previous build results to only recheck changed files.
- **skipLibCheck**: Skip type checking of `.d.ts` files to save time.
- **generateTrace**: A diagnostic flag that produces a Chrome DevTools-compatible trace of compilation.

## Real World Context
A monorepo with 200,000 lines of TypeScript can take 2+ minutes for a full type check. Incremental compilation reduces this to seconds. `skipLibCheck` saves additional time by trusting third-party types. `generateTrace` helps identify which files or types are the bottleneck.

## Deep Dive
### Incremental Compilation

```json
{
    "compilerOptions": {
        "incremental": true,
        "tsBuildInfoFile": "./dist/.tsbuildinfo"
    }
}
```

On the first build, TypeScript generates a `.tsbuildinfo` file with checksums for every source file. On subsequent builds, it only rechecks files whose checksums changed and their dependents. This typically reduces build times by 70-90%.

### skipLibCheck

```json
{
    "compilerOptions": {
        "skipLibCheck": true
    }
}
```

This skips type checking of all `.d.ts` files (including `node_modules/@types`). It is safe because:
1. Library types are already checked by their authors
2. Any type errors in `.d.ts` files are the library's responsibility
3. Your code that uses the library is still fully checked

### generateTrace for Profiling

```bash
tsc --generateTrace ./trace-output
```

This creates a `trace.json` file you can open in Chrome DevTools (Performance tab) or [perfetto.dev](https://perfetto.dev). It shows:
- Which files take the longest to check
- Which types are most expensive to resolve
- Where the compiler spends time in constraint solving

### Project Structure Optimization

- **Avoid barrel files** — A single `index.ts` that re-exports everything forces TypeScript to process all exports even if only one is used.
- **Use project references** — Split large projects into sub-projects that compile independently.
- **Reduce type complexity** — Deeply nested conditional types and recursive types slow the compiler. Simplify when possible.

```typescript
// Bad: barrel file forces loading everything
export * from "./users";
export * from "./products";
export * from "./orders";

// Better: import directly from the module
import { User } from "./users/types";
```

## Common Pitfalls
1. **Disabling skipLibCheck in CI** — Some teams disable skipLibCheck in CI for "extra safety". This adds significant build time with little benefit. Library types rarely have errors that affect your code.
2. **Ignoring .tsbuildinfo** — If `.tsbuildinfo` files are deleted (e.g., by clean scripts), every build becomes a full rebuild. Preserve them between builds.

## Best Practices
1. **Always enable incremental and skipLibCheck** — These are free performance wins with virtually no downsides.
2. **Profile before optimizing** — Use `generateTrace` to find actual bottlenecks instead of guessing.

## Summary
- Incremental compilation reuses previous results to skip unchanged files.
- `skipLibCheck` skips checking `.d.ts` files for a significant speed boost.
- `generateTrace` produces a profiling trace for identifying compilation bottlenecks.
- Avoid barrel files and deeply nested types to reduce compiler workload.

## Code Examples

**Enabling incremental compilation and skipLibCheck — two flags that dramatically reduce build times with no downside**

```json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo",
    "skipLibCheck": true,
    "declaration": true
  }
}

// First build: ~30 seconds (full)
// Second build (1 file changed): ~2 seconds (incremental)
// With skipLibCheck OFF: +5 seconds checking node_modules types
```


## Resources

- [TypeScript Performance Wiki](https://github.com/microsoft/TypeScript/wiki/Performance) — Official TypeScript wiki page on performance tips and profiling

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*