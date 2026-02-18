---
source_course: "typescript-production"
source_lesson: "typescript-production-compiler-options"
---

# The `tsconfig.json` File

This file is the root of your TypeScript project. It controls how the compiler works.

## Strictness Flags

To get the most out of TypeScript, you should enable strict mode.

- `strict`: Enables all strict type checking options. Recommended: `true`.
- `noImplicitAny`: Raises error on expressions and declarations with an implied `any` type.
- `strictNullChecks`: Makes `null` and `undefined` distinct types. Without this, `null` can be assigned to anything (like a number), leading to runtime crashes.

## Emit Options

- `target`: The JS version to output (e.g., `ES2020`).
- `module`: The module system to use (e.g., `CommonJS` for old Node, `ESNext` for bundlers).
- `outDir`: Where to place output files (usually `./dist` or `./build`).

### Resources
- [TypeScript Handbook: TSConfig Reference](https://www.typescriptlang.org/tsconfig)

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*