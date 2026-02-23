---
source_course: "typescript-production"
source_lesson: "typescript-production-monorepo-strategies"
---

# Monorepo Strategies with TypeScript

## Introduction
Modern monorepo tools like Nx and Turborepo orchestrate TypeScript compilation across dozens of packages. Understanding how they integrate with TypeScript's project references, shared configs, and build caching helps you get fast builds and consistent type checking across your entire monorepo.

## Key Concepts
- **Nx**: A monorepo toolkit that provides project graph analysis, task caching, and TypeScript integration.
- **Turborepo**: A build system for monorepos that caches task outputs and runs tasks in parallel.
- **Internal Packages**: Packages within a monorepo that are consumed by other packages but not published to npm.

## Real World Context
A startup monorepo might have 20+ packages: shared types, UI components, API services, and multiple apps. Without proper tooling, changing a shared type requires manually rebuilding every dependent package. Nx/Turborepo automate this dependency tracking.

## Deep Dive
### Nx with TypeScript

Nx uses its project graph to determine which packages need rebuilding:

```json
// nx.json
{
    "targetDefaults": {
        "build": {
            "dependsOn": ["^build"],
            "cache": true
        },
        "typecheck": {
            "dependsOn": ["^typecheck"],
            "cache": true
        }
    }
}
```

The `^build` syntax means "build dependencies first". Nx caches the output so repeated builds are instant.

### Internal Package Pattern

For packages consumed only within the monorepo (not published), you can skip the build step entirely using TypeScript path aliases:

```json
// tsconfig.base.json
{
    "compilerOptions": {
        "paths": {
            "@myorg/shared": ["libs/shared/src/index.ts"],
            "@myorg/ui": ["libs/ui/src/index.ts"]
        }
    }
}
```

TypeScript resolves `@myorg/shared` directly to the source `.ts` file — no build step needed for development. Build only for production or publishing.

### Turborepo Configuration

```json
// turbo.json
{
    "pipeline": {
        "build": {
            "dependsOn": ["^build"],
            "outputs": ["dist/**"]
        },
        "typecheck": {
            "dependsOn": ["^build"]
        }
    }
}
```

Turborepo caches build outputs based on file content hashes, so only changed packages get rebuilt.

### Shared TSConfig

```json
// packages/tsconfig/base.json (shared config package)
{
    "compilerOptions": {
        "strict": true,
        "target": "ES2022",
        "module": "NodeNext",
        "moduleResolution": "NodeNext"
    }
}
```

Each package extends this shared config, ensuring consistency.

## Common Pitfalls
1. **Circular dependencies** — Package A imports from B, and B imports from A. Monorepo tools detect this, but fixing it requires refactoring shared code into a third package.
2. **Inconsistent TypeScript versions** — Different packages using different TS versions causes subtle type incompatibilities. Pin a single version in the root package.json.

## Best Practices
1. **Use internal packages for development** — Skip build steps by pointing imports to source via `paths`. Build only for production.
2. **Pin TypeScript version at the root** — Use a single TypeScript version across the entire monorepo.

## Summary
- Nx and Turborepo orchestrate TypeScript builds with dependency tracking and caching.
- Internal packages can use path aliases to skip build steps during development.
- Shared tsconfig packages ensure consistent compiler options across all packages.
- Pin TypeScript to a single version to avoid cross-package type incompatibilities.

## Code Examples

**Internal package pattern — TypeScript resolves directly to source files via path aliases, skipping the build step**

```json
// Internal package — no build step needed
// tsconfig.base.json
{
  "compilerOptions": {
    "paths": {
      "@myorg/shared": ["libs/shared/src/index.ts"],
      "@myorg/ui": ["libs/ui/src/index.ts"]
    }
  }
}

// package.json of libs/shared
{
  "name": "@myorg/shared",
  "main": "src/index.ts",
  "types": "src/index.ts"
}
```


## Resources

- [Nx Documentation](https://nx.dev/getting-started/intro) — Official Nx documentation for monorepo management

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*