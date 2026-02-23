---
source_course: "typescript-production"
source_lesson: "typescript-production-strict-family"
---

# The Strict Flag Family

## Introduction
The `strict: true` flag is not a single check — it is an umbrella that enables 9 individual sub-flags. Understanding each one helps you know exactly what protections you are getting and lets you selectively disable specific checks during migration.

## Key Concepts
- **strict**: A master flag that enables all strict sub-flags at once.
- **Sub-flags**: Individual flags that each enforce a specific type-checking rule.
- **Incremental Adoption**: You can enable sub-flags one at a time when migrating an existing project to strict mode.

## Real World Context
When migrating a large JavaScript codebase to TypeScript, enabling `strict: true` all at once may produce thousands of errors. Understanding the sub-flags lets you enable them incrementally — start with `noImplicitAny`, fix those errors, then enable `strictNullChecks`, and so on.

## Deep Dive
### The 9 Strict Sub-Flags

As of TypeScript 5.9, `strict: true` enables these individual flags:

1. **strictNullChecks** — `null` and `undefined` are distinct types. `let x: string = null` is an error.

```typescript
// Without strictNullChecks
let name: string = null; // OK (dangerous!)

// With strictNullChecks
let name: string = null; // Error: Type 'null' is not assignable to type 'string'
let maybeName: string | null = null; // OK — explicitly nullable
```

2. **noImplicitAny** — Error when TypeScript cannot infer a type and falls back to `any`.

```typescript
// Error: Parameter 'x' implicitly has an 'any' type
function double(x) { return x * 2; }

// Fix: annotate the parameter
function double(x: number) { return x * 2; }
```

3. **strictFunctionTypes** — Function parameter types are checked contravariantly (stricter than the default bivariant check).

4. **strictBindCallApply** — `bind`, `call`, and `apply` are fully type-checked instead of returning `any`.

5. **strictPropertyInitialization** — Class properties must be initialized in the constructor or have a definite assignment assertion (`!`).

```typescript
class User {
    name: string; // Error: not initialized
    constructor() {} // Missing this.name = ...
}
```

6. **alwaysStrict** — Emits `"use strict"` at the top of every output file.

7. **useUnknownInCatchVariables** — Catch clause variables are typed as `unknown` instead of `any`.

```typescript
try { /* ... */ } catch (e) {
    // e is 'unknown', not 'any'
    if (e instanceof Error) console.log(e.message);
}
```

8. **noImplicitThis** — Error when `this` has an implicit `any` type.

9. **strictBuiltinIteratorReturn** — Built-in iterators return `{ value: T, done: boolean }` with stricter typing for the `done` state.

### Incremental Migration Strategy

```json
{
    "compilerOptions": {
        "strict": false,
        "noImplicitAny": true,
        "strictNullChecks": true
        // Enable more flags as you fix errors
    }
}
```

## Common Pitfalls
1. **Assuming strict is just one check** — Developers sometimes disable strict thinking it is a single feature. It actually controls 9 separate safety features.
2. **Using `!` to silence strictPropertyInitialization** — The definite assignment assertion (`name!: string`) bypasses the check. Use it sparingly and only when you know the property will be set before access.

## Best Practices
1. **Enable strict: true on new projects** — There is no reason to start without it. The short-term friction saves long-term debugging.
2. **Migrate incrementally** — For existing projects, enable sub-flags one at a time, fix errors, commit, and repeat.

## Summary
- `strict: true` enables 9 individual type-checking sub-flags.
- Key sub-flags include `strictNullChecks`, `noImplicitAny`, and `useUnknownInCatchVariables`.
- For migration, enable sub-flags incrementally instead of all at once.
- Never use `!` (definite assignment) as a blanket fix for `strictPropertyInitialization`.

## Code Examples

**With useUnknownInCatchVariables, catch variables are typed as unknown instead of any — forcing explicit type narrowing**

```typescript
// useUnknownInCatchVariables: catch variable is 'unknown'
try {
    JSON.parse("invalid json");
} catch (error) {
    // error is 'unknown' — must narrow before using
    if (error instanceof SyntaxError) {
        console.log(error.message); // Safe: error is SyntaxError
    } else {
        console.log("Unexpected error", error);
    }
}

// Without this flag, error would be 'any' — no type checking at all
```


## Resources

- [TypeScript Handbook: Strict Mode](https://www.typescriptlang.org/tsconfig#strict) — Official reference for the strict flag and all its sub-flags

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*