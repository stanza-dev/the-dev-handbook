---
source_course: "typescript-production"
source_lesson: "typescript-production-migration-strategy"
---

# JavaScript to TypeScript Migration Strategy

## Introduction
Migrating a JavaScript project to TypeScript is not an all-or-nothing decision. A phased approach lets you add type safety incrementally while keeping the project functional at every stage. This lesson covers the proven migration strategies used by teams at companies like Airbnb, Slack, and Bloomberg.

## Key Concepts
- **Incremental Migration**: Converting files one at a time from `.js` to `.ts`, starting with the most critical modules.
- **allowJs**: A tsconfig flag that lets TypeScript compile `.js` files alongside `.ts` files.
- **Strict Graduation**: Starting with `strict: false` and enabling individual strict flags as the codebase becomes more typed.

## Real World Context
Airbnb migrated millions of lines from JavaScript to TypeScript over two years. They used automated codemods to add basic type annotations, then gradually tightened strictness. The key insight: you do not need 100% type coverage to get significant value from TypeScript.

## Deep Dive
### Phase 1: Setup (Day 1)

Enable TypeScript alongside JavaScript:

```json
{
    "compilerOptions": {
        "allowJs": true,
        "checkJs": false,
        "strict": false,
        "noEmit": true,
        "target": "ES2022",
        "module": "NodeNext",
        "moduleResolution": "NodeNext"
    },
    "include": ["src/**/*"]
}
```

With `allowJs: true` and `noEmit: true`, TypeScript checks `.ts` files but passes through `.js` files unchanged.

### Phase 2: Convert Critical Files

Rename high-value files from `.js` to `.ts`:
1. Shared utilities (used by many modules)
2. Data models and API types
3. Authentication and payment logic

```typescript
// src/models/user.ts (renamed from .js)
export interface User {
    id: string;
    name: string;
    email: string;
    role: "admin" | "user";
}
```

### Phase 3: Enable checkJs

Turn on type checking for remaining `.js` files:

```json
{
    "compilerOptions": {
        "checkJs": true
    }
}
```

Add `@ts-check` to individual files or use `checkJs` globally. Fix errors file by file.

### Phase 4: Strict Graduation

Enable strict sub-flags one at a time:

```json
{
    "compilerOptions": {
        "strict": false,
        "noImplicitAny": true,
        "strictNullChecks": true
        // Add more flags as you fix errors
    }
}
```

### Phase 5: Full Strict

Once all files are `.ts` and all errors are fixed:

```json
{
    "compilerOptions": {
        "strict": true,
        "allowJs": false
    }
}
```

## Common Pitfalls
1. **Big-bang migration** — Converting all files at once leads to thousands of errors and team frustration. Always migrate incrementally.
2. **Using `any` as a crutch** — It is tempting to type everything as `any` to make errors go away. This defeats the purpose. Use `unknown` and narrow types instead.

## Best Practices
1. **Start with shared types** — Define interfaces for your domain models first. These types propagate through the codebase as you convert files.
2. **Track progress with metrics** — Count the percentage of `.ts` files and the number of `any` types. Set quarterly goals for improvement.

## Summary
- Migrate incrementally: setup → convert critical files → checkJs → strict graduation → full strict.
- `allowJs: true` lets TypeScript and JavaScript coexist during migration.
- Start with shared utilities and domain models for maximum type propagation.
- Track migration progress with metrics and set realistic timelines.

## Code Examples

**Migration tsconfig progression — start with allowJs and no strictness, graduate to full strict mode**

```json
// Phase 1: Initial tsconfig for migration
{
  "compilerOptions": {
    "allowJs": true,
    "checkJs": false,
    "strict": false,
    "noEmit": true,
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "esModuleInterop": true
  },
  "include": ["src/**/*"]
}

// Phase 5: Final strict tsconfig
// {
//   "compilerOptions": {
//     "allowJs": false,
//     "strict": true,
//     ...
//   }
// }
```


## Resources

- [TypeScript: Migrating from JavaScript](https://www.typescriptlang.org/docs/handbook/migrating-from-javascript.html) — Official TypeScript migration guide from JavaScript

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*