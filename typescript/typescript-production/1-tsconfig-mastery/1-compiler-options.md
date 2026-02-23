---
source_course: "typescript-production"
source_lesson: "typescript-production-compiler-options"
---

# Essential Compiler Options

## Introduction
The `tsconfig.json` file is the control center of every TypeScript project. It tells the compiler which files to include, how strictly to check types, what JavaScript version to output, and how to resolve modules. Understanding its options is essential for setting up reliable TypeScript projects.

## Key Concepts
- **tsconfig.json**: A JSON configuration file at the root of a TypeScript project that specifies compiler behavior.
- **Strict Mode**: A master flag that enables all strict type-checking sub-flags at once.
- **Target**: The ECMAScript version for the compiled JavaScript output.
- **Module**: The module system used in the output (CommonJS, ESNext, NodeNext).

## Real World Context
When you run `tsc --init` in TypeScript 5.9, it generates a tsconfig with `strict: true` enabled by default. Every option you set (or leave at default) affects build output, type safety, and compatibility. Getting the configuration right from day one avoids painful migrations later.

## Deep Dive
### Creating a tsconfig.json

```bash
npx tsc --init
```

This generates a starter configuration. A recommended modern setup:

```json
{
    "compilerOptions": {
        "target": "ES2022",
        "module": "NodeNext",
        "moduleResolution": "NodeNext",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "outDir": "./dist",
        "rootDir": "./src",
        "declaration": true,
        "declarationMap": true,
        "sourceMap": true
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "dist"]
}
```

### Key Options Explained

**Type Checking:**
- `strict`: Enables all strict sub-flags (covered in the next lesson).
- `noUncheckedIndexedAccess`: Makes array/object index access return `T | undefined` instead of `T`.
- `exactOptionalProperties`: Distinguishes between missing properties and properties set to `undefined`.

**Emit:**
- `target`: The JS version to output. `ES2022` supports top-level await, `Array.at()`, and class fields.
- `module`: The module system. `NodeNext` for Node.js, `ESNext` for bundlers.
- `outDir` / `rootDir`: Control where compiled files go and where source files live.
- `declaration`: Generate `.d.ts` files for library consumers.
- `sourceMap`: Generate `.js.map` files for debugging.

**Module Resolution:**
- `moduleResolution`: How TypeScript finds imported files. `NodeNext` for Node.js ESM, `Bundler` for Vite/Webpack.
- `paths`: Define import aliases like `@utils/*`.

### The include and exclude Fields

```json
{
    "include": ["src/**/*"],
    "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

`include` specifies which files to compile. `exclude` removes files from the inclusion set. `node_modules` is excluded by default.

## Common Pitfalls
1. **Forgetting `moduleResolution`** — Setting `module` without a matching `moduleResolution` leads to confusing import errors. Use `NodeNext` / `NodeNext` or `ESNext` / `Bundler` as pairs.
2. **Setting target too low** — `ES5` output dramatically increases bundle size because modern features must be polyfilled. Use the highest target your runtime supports.

## Best Practices
1. **Start with strict: true** — Retrofitting strict mode onto an existing project requires fixing hundreds of errors. Enable it from the start.
2. **Use declaration and declarationMap for libraries** — Consumers need `.d.ts` files for type information and `.d.ts.map` files for "Go to Definition" to show source code.

## Summary
- `tsconfig.json` controls compilation, type checking, and module resolution.
- `strict: true` enables comprehensive type safety and should always be on.
- Match `module` and `moduleResolution` as a pair (e.g., NodeNext/NodeNext).
- Use `declaration` and `sourceMap` for libraries and debugging.

## Code Examples

**A recommended tsconfig.json for a modern Node.js project in TypeScript 5.9 with strict mode and NodeNext modules**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "sourceMap": true
  },
  "include": ["src/**/*"]
}
```


## Resources

- [TSConfig Reference](https://www.typescriptlang.org/tsconfig) — Complete reference for every tsconfig.json option
- [What is a tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) — Official guide to the TypeScript configuration file

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*