---
source_course: "typescript-production"
source_lesson: "typescript-production-tsconfig-extends"
---

# TSConfig Inheritance & Shared Bases

## Introduction
Large projects and monorepos need multiple tsconfig files — one per package, one for tests, one for builds. TSConfig inheritance (`extends`) lets you define shared settings in a base file and override only what differs in each consumer, eliminating duplication and ensuring consistency.

## Key Concepts
- **extends**: A tsconfig property that inherits all options from a base configuration file.
- **@tsconfig/bases**: A collection of community-maintained base tsconfigs for common environments (Node 20, strictest, etc.).
- **Override Semantics**: Consumer options override base options. Array options like `lib` are replaced, not merged.

## Real World Context
In an Nx monorepo, you might have a root `tsconfig.base.json` with shared strict settings, then per-app configs that override `outDir`, `rootDir`, and `paths`. The `@tsconfig/strictest` base is popular for projects that want maximum type safety beyond the default `strict: true`.

## Deep Dive
### Basic Inheritance

```json
// tsconfig.base.json
{
    "compilerOptions": {
        "strict": true,
        "target": "ES2022",
        "module": "NodeNext",
        "moduleResolution": "NodeNext",
        "esModuleInterop": true,
        "skipLibCheck": true,
        "declaration": true
    }
}
```

```json
// apps/api/tsconfig.json
{
    "extends": "../../tsconfig.base.json",
    "compilerOptions": {
        "outDir": "./dist",
        "rootDir": "./src"
    },
    "include": ["src/**/*"]
}
```

The API config inherits all base options and only specifies its own `outDir`, `rootDir`, and `include`.

### Using @tsconfig/bases

```bash
npm install -D @tsconfig/node20 @tsconfig/strictest
```

```json
{
    "extends": "@tsconfig/node20/tsconfig.json",
    "compilerOptions": {
        "outDir": "./dist"
    }
}
```

Popular bases:
- `@tsconfig/node20` — Recommended settings for Node.js 20
- `@tsconfig/strictest` — Maximum strictness (adds `noUncheckedIndexedAccess`, `exactOptionalProperties`, etc.)

### Multiple Extends (TypeScript 5.0+)

TypeScript 5.0+ supports extending from multiple bases using an array:

```json
{
    "extends": [
        "@tsconfig/node20/tsconfig.json",
        "@tsconfig/strictest/tsconfig.json"
    ],
    "compilerOptions": {
        "outDir": "./dist"
    }
}
```

Later entries in the array take precedence over earlier ones for conflicting options.

### Override Semantics

Important: array properties like `lib` and `types` are **replaced**, not merged:

```json
// base: { "lib": ["ES2022", "DOM"] }
// consumer: { "lib": ["ES2022"] }
// Result: ["ES2022"] — DOM is NOT included
```

## Common Pitfalls
1. **Assuming arrays merge** — `lib`, `types`, and `include` from the base are replaced entirely if the consumer specifies them. You must repeat all entries you want to keep.
2. **Relative path confusion** — The `extends` path is relative to the file containing it, but `include`/`exclude` paths are relative to the consuming tsconfig.

## Best Practices
1. **Use a monorepo base config** — Create a `tsconfig.base.json` at the root with shared settings. Per-app configs extend it.
2. **Use @tsconfig/strictest for new projects** — It adds safety flags beyond `strict: true` that catch additional edge cases.

## Summary
- `extends` inherits options from a base tsconfig, reducing duplication.
- `@tsconfig/bases` provides community-maintained configs for common environments.
- TypeScript 5.0+ supports multiple extends via an array.
- Array properties are replaced, not merged — repeat all values you need.

## Code Examples

**Monorepo pattern — a shared base config with strict settings inherited by per-package configs that override only paths**

```json
// tsconfig.base.json — shared across all packages
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  }
}

// packages/api/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```


## Resources

- [TSConfig extends Reference](https://www.typescriptlang.org/tsconfig#extends) — Official docs on the extends option and inheritance semantics
- [@tsconfig/bases on GitHub](https://github.com/tsconfig/bases) — Community-maintained TSConfig base configurations for various environments

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*