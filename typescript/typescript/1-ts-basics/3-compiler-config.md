---
source_course: "typescript"
source_lesson: "typescript-compiler-config"
---

# Compiler Configuration

## Introduction
The TypeScript compiler (`tsc`) transforms your `.ts` files into `.js` files that browsers and Node.js can run. The `tsconfig.json` file controls how the compiler behaves, from which files to include to how strict the type checking should be.

## Key Concepts
- **tsconfig.json**: A JSON configuration file at the root of a TypeScript project that specifies compiler options and file inclusion rules.
- **Strict Mode**: A collection of compiler flags that enable the strictest level of type checking.
- **Target**: The JavaScript version the compiler outputs (e.g., ES2020, ES2022).
- **Module**: The module system used in the output (e.g., CommonJS, ESNext).

## Real World Context
When you join a new team, the `tsconfig.json` is one of the first files to review. It tells you the project's type-checking strictness, which JavaScript features are available, and how modules are resolved. Misconfiguring it leads to confusing errors or silently allowing unsafe code.

## Deep Dive

### Creating a tsconfig.json

You can generate a starter configuration with:

```bash
npx tsc --init
```

This creates a `tsconfig.json` with commented-out options. A minimal configuration looks like this:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```

Each option serves a purpose: `target` sets the output JavaScript version, `module` sets the module system, `strict` turns on all strict checks, `outDir` is where compiled files go, and `rootDir` is where source files live.

### The strict Flag

The `strict` flag is actually a shorthand that enables multiple individual flags at once:

```typescript
// These are all enabled by "strict": true
// strictNullChecks — null and undefined are distinct types
// noImplicitAny — error when a type cannot be inferred
// strictFunctionTypes — stricter function parameter checking
// strictBindCallApply — check bind, call, apply arguments
// strictPropertyInitialization — class properties must be initialized
// alwaysStrict — emit "use strict" in output files
```

With `strictNullChecks` enabled, you cannot assign `null` to a `string` variable without explicitly allowing it:

```typescript
let name: string = "Alice";
// name = null; // Error: Type 'null' is not assignable to type 'string'
let maybeName: string | null = null; // OK — explicitly nullable
```

This prevents an entire class of "null reference" bugs.

### Target and Module

The `target` option determines which JavaScript features are available in the output. Setting `"target": "ES2022"` means you can use top-level `await`, `Array.at()`, and other ES2022 features directly.

The `module` option controls the import/export syntax in the output. For modern projects, `"ESNext"` or `"NodeNext"` are recommended.

## Common Pitfalls
1. **Starting without strict mode** — It is tempting to disable strict mode to avoid errors, but this allows unsafe code to creep in. Enabling strict mode later requires fixing many existing issues.
2. **Mismatching target and runtime** — Setting `target` to `ES2022` but running on an old Node.js version that does not support ES2022 features causes runtime errors.

## Best Practices
1. **Always enable strict mode** — Start every new project with `"strict": true`. The short-term friction pays off in long-term safety.
2. **Match target to your deployment environment** — Check which ECMAScript version your runtime supports and set `target` accordingly.

## Summary
- `tsconfig.json` controls the TypeScript compiler's behavior.
- The `strict` flag enables comprehensive type checking and should always be turned on.
- `target` and `module` must match your deployment runtime to avoid compatibility issues.

## Code Examples

**A recommended tsconfig.json for a modern TypeScript project with strict mode enabled and ES2022 target**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```


## Resources

- [TSConfig Reference](https://www.typescriptlang.org/tsconfig) — Complete reference for every tsconfig.json option
- [What is a tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) — Official guide to the TypeScript configuration file

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*