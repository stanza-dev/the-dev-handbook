---
source_course: "typescript-production"
source_lesson: "typescript-production-project-references"
---

# Project References & Composite

## Introduction
As TypeScript projects grow, compilation slows because the compiler must process every file. Project references split a large project into smaller sub-projects that can be compiled independently and incrementally. This dramatically reduces build times in monorepos and multi-package setups.

## Key Concepts
- **Project Reference**: A tsconfig entry pointing to another sub-project, establishing a dependency relationship.
- **composite**: A tsconfig flag that marks a project as a referenceable sub-project. It requires `declaration: true` and enables incremental builds.
- **tsc --build (-b)**: A special build mode that compiles projects in dependency order, skipping up-to-date ones.

## Real World Context
In a monorepo with `libs/shared`, `apps/api`, and `apps/web`, the API and web apps both depend on shared. Without project references, changing a file in shared requires recompiling everything. With references, `tsc --build` recompiles only shared and its downstream dependents.

## Deep Dive
### Setting Up Composite

Every referenced project must have `composite: true`:

```json
// libs/shared/tsconfig.json
{
    "compilerOptions": {
        "composite": true,
        "declaration": true,
        "declarationMap": true,
        "outDir": "./dist",
        "rootDir": "./src"
    },
    "include": ["src/**/*"]
}
```

`composite: true` enforces three things:
1. `declaration` must be true (generates `.d.ts` files)
2. All source files must be matched by `include` or `files`
3. Incremental build metadata is stored (`.tsbuildinfo` files)

### Adding References

The consuming project references its dependencies:

```json
// apps/api/tsconfig.json
{
    "compilerOptions": {
        "outDir": "./dist",
        "rootDir": "./src"
    },
    "references": [
        { "path": "../../libs/shared" }
    ],
    "include": ["src/**/*"]
}
```

### Root Build Configuration

A root tsconfig ties everything together:

```json
// tsconfig.json (root)
{
    "files": [],
    "references": [
        { "path": "./libs/shared" },
        { "path": "./apps/api" },
        { "path": "./apps/web" }
    ]
}
```

### Building

```bash
tsc --build          # Build all projects in dependency order
tsc --build --watch  # Watch mode across all projects
tsc --build --clean  # Remove all build outputs
```

`tsc --build` is smart: it checks `.tsbuildinfo` files and only recompiles projects whose source files have changed.

## Common Pitfalls
1. **Forgetting composite: true** — Without it, the project cannot be referenced and `tsc --build` will fail with a confusing error.
2. **Importing source instead of declarations** — Referenced projects should be imported by their package name or path alias, not by relative path to source files.

## Best Practices
1. **Use --build for all monorepo compilation** — Never use plain `tsc` on individual packages in a monorepo. `tsc --build` respects the dependency graph.
2. **Enable declarationMap** — It lets "Go to Definition" navigate to source files instead of `.d.ts` files.

## Summary
- Project references split large projects into independently compilable sub-projects.
- `composite: true` is required for any referenced project and enables incremental builds.
- `tsc --build` compiles in dependency order and skips up-to-date projects.
- Root tsconfig with `"files": []` and `"references"` ties the monorepo together.

## Code Examples

**A monorepo setup with root tsconfig referencing sub-projects — each lib/app has composite: true for incremental builds**

```json
// Root tsconfig.json for a monorepo
{
  "files": [],
  "references": [
    { "path": "./libs/shared" },
    { "path": "./libs/database" },
    { "path": "./apps/api" },
    { "path": "./apps/web" }
  ]
}

// libs/shared/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```


## Resources

- [TypeScript: Project References](https://www.typescriptlang.org/docs/handbook/project-references.html) — Official handbook on setting up and using project references

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*