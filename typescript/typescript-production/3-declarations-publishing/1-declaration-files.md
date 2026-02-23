---
source_course: "typescript-production"
source_lesson: "typescript-production-declaration-files"
---

# Understanding .d.ts Files

## Introduction
Declaration files (`.d.ts`) describe the shape of JavaScript code without containing any implementation. They are the bridge between TypeScript's type system and the vast ecosystem of untyped JavaScript libraries. Understanding how they work is essential for consuming third-party packages and publishing your own typed libraries.

## Key Concepts
- **Declaration File (.d.ts)**: A file containing only type information — no runtime code. It tells TypeScript what types a JavaScript module exports.
- **Ambient Declaration**: A `declare` statement that tells TypeScript about values that exist at runtime but are not defined in the current file.
- **DefinitelyTyped (@types)**: A community repository of declaration files for popular JavaScript libraries.

## Real World Context
When you import `lodash` in a TypeScript project, TypeScript needs to know what functions lodash exports and what types they accept. If lodash does not ship its own types, you install `@types/lodash` from DefinitelyTyped. These `@types` packages contain `.d.ts` files that describe lodash's API.

## Deep Dive
### How TypeScript Finds Declarations

When you write `import _ from "lodash"`, TypeScript looks for types in this order:
1. `lodash/index.d.ts` (bundled with the package)
2. `@types/lodash/index.d.ts` (from DefinitelyTyped)
3. A `paths` mapping in your tsconfig

### Generating Declaration Files

Enable `declaration` in tsconfig to generate `.d.ts` files from your source:

```json
{
    "compilerOptions": {
        "declaration": true,
        "declarationMap": true,
        "emitDeclarationOnly": false,
        "outDir": "./dist"
    }
}
```

For a source file `src/utils.ts`:
```typescript
export function add(a: number, b: number): number {
    return a + b;
}
```

TypeScript generates `dist/utils.d.ts`:
```typescript
export declare function add(a: number, b: number): number;
```

### Ambient Declarations

Use `declare` for values injected by the runtime environment:

```typescript
// globals.d.ts
declare const __APP_VERSION__: string;
declare const __DEV__: boolean;

// Usage in any .ts file
console.log(`Version: ${__APP_VERSION__}`);
if (__DEV__) { enableDebugMode(); }
```

### Triple-Slash Directives

Reference additional type files:

```typescript
/// <reference types="node" />
/// <reference path="./custom-types.d.ts" />
```

## Common Pitfalls
1. **Forgetting declarationMap** — Without `.d.ts.map` files, "Go to Definition" in your IDE jumps to the `.d.ts` file instead of the source `.ts` file. Always enable `declarationMap`.
2. **Conflicting ambient declarations** — Two `.d.ts` files declaring the same global can cause errors. Use module-scoped declarations when possible.

## Best Practices
1. **Enable declaration + declarationMap for libraries** — Consumers get types and can navigate to source code.
2. **Use @types only as a fallback** — Prefer packages that bundle their own types. Bundled types are always version-matched.

## Summary
- Declaration files (`.d.ts`) provide type information for JavaScript code.
- TypeScript generates them automatically with `declaration: true`.
- `declare` creates ambient declarations for runtime globals.
- DefinitelyTyped (`@types`) provides types for packages that don't bundle their own.

## Code Examples

**Ambient declarations for globals and typed environment variables — TypeScript knows these values exist at runtime**

```typescript
// globals.d.ts — declare runtime globals
declare const __APP_VERSION__: string;
declare const __BUILD_TIME__: number;

// env.d.ts — type environment variables
declare namespace NodeJS {
    interface ProcessEnv {
        NODE_ENV: "development" | "production" | "test";
        DATABASE_URL: string;
        API_KEY?: string;
    }
}

// Usage — TypeScript knows these exist
console.log(__APP_VERSION__);
console.log(process.env.DATABASE_URL); // string, not string | undefined
```


## Resources

- [TypeScript Handbook: Declaration Files](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html) — Official guide to creating and using declaration files

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*