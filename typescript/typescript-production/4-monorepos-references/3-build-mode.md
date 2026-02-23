---
source_course: "typescript-production"
source_lesson: "typescript-production-build-mode"
---

# tsc --build Mode

## Introduction
The `tsc --build` (or `tsc -b`) command is purpose-built for multi-project TypeScript setups. Unlike plain `tsc`, it understands project references, builds dependencies before dependents, and uses `.tsbuildinfo` files for incremental compilation. It is the correct way to compile TypeScript monorepos.

## Key Concepts
- **tsc --build (-b)**: A compilation mode that respects project references and builds in dependency order.
- **.tsbuildinfo**: A file storing incremental compilation state, allowing the compiler to skip unchanged files.
- **Incremental Compilation**: Only recompiling files that changed since the last build.

## Real World Context
In a monorepo with 50 packages, running `tsc` on each package individually is slow and error-prone (wrong order, stale declarations). `tsc --build` handles the dependency graph automatically and only recompiles what changed.

## Deep Dive
### Basic Usage

```bash
# Build the current project and all its references
tsc --build

# Build a specific project
tsc --build apps/api/tsconfig.json

# Watch mode — rebuild on changes across all projects
tsc --build --watch

# Clean all build outputs
tsc --build --clean

# Force rebuild (ignore .tsbuildinfo)
tsc --build --force
```

### How Incremental Works

When `composite: true` or `incremental: true` is set, TypeScript writes a `.tsbuildinfo` file after compilation:

```json
{
    "compilerOptions": {
        "composite": true,
        "incremental": true,
        "tsBuildInfoFile": "./dist/.tsbuildinfo"
    }
}
```

On subsequent builds, TypeScript reads this file and skips files whose content hash has not changed. This can reduce build times by 80-90% for large projects.

### Build Order

Given references:
```
shared → database → api
shared → ui → web
```

`tsc --build` compiles in topological order: `shared` first (no dependencies), then `database` and `ui` (depend on shared), then `api` and `web` (depend on database/ui respectively).

### Verbose Output

```bash
tsc --build --verbose
# Output:
# Project 'libs/shared' is up to date
# Building project 'apps/api'...
# Project 'apps/web' is up to date
```

## Common Pitfalls
1. **Using plain tsc in a monorepo** — Plain `tsc` does not understand references. It compiles a single project and may use stale `.d.ts` files from dependencies.
2. **Stale .tsbuildinfo files** — If you manually move or delete files, the `.tsbuildinfo` cache may be stale. Use `tsc --build --force` to do a clean rebuild.

## Best Practices
1. **Always use tsc --build for monorepos** — It is the only mode that correctly handles project references.
2. **Add .tsbuildinfo to .gitignore** — These files are machine-specific build caches and should not be committed.

## Summary
- `tsc --build` compiles projects in dependency order using project references.
- `.tsbuildinfo` files enable incremental compilation, skipping unchanged files.
- Use `--watch` for development and `--clean` to remove outputs.
- Always use `tsc --build` instead of plain `tsc` in multi-project setups.

## Code Examples

**Essential tsc --build commands for monorepo development — from basic builds to watch mode and cache management**

```bash
# Common tsc --build commands

# Build all projects from root
tsc --build

# Build with verbose output to see what's happening
tsc --build --verbose

# Watch mode — recompile on any file change
tsc --build --watch

# Clean all outputs (.js, .d.ts, .tsbuildinfo)
tsc --build --clean

# Force full rebuild, ignoring cached .tsbuildinfo
tsc --build --force

# Build a specific project and its dependencies
tsc --build apps/api/tsconfig.json
```


## Resources

- [TypeScript: tsc CLI Reference](https://www.typescriptlang.org/docs/handbook/compiler-options.html) — Official reference for TypeScript compiler CLI options including --build mode

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*