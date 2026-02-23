---
source_course: "typescript-production"
source_lesson: "typescript-production-bundler-integration"
---

# Bundler Integration & isolatedModules

## Introduction
Modern bundlers like esbuild, swc, and tsup compile TypeScript orders of magnitude faster than `tsc` by skipping type checking entirely — they only strip types. The `isolatedModules` flag ensures your code is compatible with these single-file transpilers by disallowing patterns that require whole-program analysis.

## Key Concepts
- **Type Stripping**: Removing TypeScript syntax (type annotations, interfaces) to produce JavaScript, without performing any type checking.
- **isolatedModules**: A tsconfig flag that ensures each file can be transpiled independently, without knowledge of other files.
- **esbuild / swc**: Extremely fast TypeScript transpilers written in Go and Rust respectively.

## Real World Context
Vite uses esbuild for development and serves TypeScript files in under 50ms. Next.js uses swc. Both tools strip types without checking them. If your code uses patterns that require cross-file type information (like `const enum` or namespace merging), these tools silently produce wrong output. `isolatedModules` catches this at compile time.

## Deep Dive
### The Two-Tool Approach

Modern projects use two tools for TypeScript:
1. **tsc** — Type checking only (`noEmit: true`)
2. **esbuild/swc/tsup** — Fast transpilation (type stripping)

```json
{
    "compilerOptions": {
        "noEmit": true,
        "isolatedModules": true,
        "strict": true
    }
}
```

### What isolatedModules Disallows

Patterns that require knowledge of other files:

```typescript
// 1. Re-exporting types without 'type' keyword
export { User } from "./types"; // Error with isolatedModules
export type { User } from "./types"; // OK — explicitly a type

// 2. const enum (values are inlined from the defining file)
const enum Direction { Up, Down } // Error with isolatedModules
enum Direction { Up, Down } // OK — regular enum

// 3. Namespace merging across files
// File A: namespace Foo { export const x = 1; }
// File B: namespace Foo { export const y = 2; }
// Error — bundlers process files independently
```

### Build Tool Comparison

| Tool | Speed | Type Checking | Language |
|------|-------|--------------|----------|
| tsc | Slow (~30s) | Full | TypeScript |
| esbuild | Fast (~0.3s) | None | Go |
| swc | Fast (~0.5s) | None | Rust |
| tsup | Fast (~1s) | Optional via tsc | Go (esbuild) |

### tsup Example

```bash
# Build with tsup (uses esbuild internally)
tsup src/index.ts --format cjs,esm --dts

# This runs esbuild for transpilation + tsc for .d.ts generation
```

## Common Pitfalls
1. **Skipping type checking entirely** — Fast bundlers do not check types. You must still run `tsc --noEmit` in CI or as a pre-commit hook.
2. **Using const enum with bundlers** — `const enum` requires cross-file analysis that bundlers cannot do. Use regular `enum` or string unions instead.

## Best Practices
1. **Always enable isolatedModules** — It ensures your code works with any transpiler. This flag is required by Vite, Next.js, and most modern frameworks.
2. **Run tsc in CI** — Use `tsc --noEmit` in your CI pipeline for type checking even if you use esbuild/swc for builds.

## Summary
- Modern bundlers (esbuild, swc) strip types without checking them — 100x faster than tsc.
- `isolatedModules` ensures code is compatible with single-file transpilers.
- Use the two-tool approach: tsc for checking, esbuild/swc for transpilation.
- Always run `tsc --noEmit` in CI to catch type errors.

## Code Examples

**isolatedModules flags patterns that break single-file transpilers — use explicit type exports and regular enums**

```typescript
// With isolatedModules: true

// BAD: re-export without 'type' — bundler can't tell if it's a value or type
export { User } from "./types"; // Error

// GOOD: explicit type re-export
export type { User } from "./types"; // OK

// BAD: const enum requires cross-file analysis
const enum Color { Red, Green, Blue } // Error

// GOOD: regular enum or string union
enum Color { Red, Green, Blue } // OK
type Color = "red" | "green" | "blue"; // Also OK
```


## Resources

- [TypeScript: isolatedModules](https://www.typescriptlang.org/tsconfig#isolatedModules) — Official reference for the isolatedModules compiler option

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*