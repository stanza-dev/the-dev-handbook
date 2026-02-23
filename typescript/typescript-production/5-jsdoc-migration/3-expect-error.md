---
source_course: "typescript-production"
source_lesson: "typescript-production-ts-ignore-expect-error"
---

# @ts-ignore vs @ts-expect-error

## Introduction
Sometimes you need to suppress a TypeScript error — during migration, for intentionally dynamic code, or when working around a library's incorrect types. TypeScript provides two directive comments for this: `@ts-ignore` and `@ts-expect-error`. They look similar but behave very differently, and choosing wrong can hide bugs.

## Key Concepts
- **@ts-ignore**: Suppresses the error on the next line. Silent even if there is no error to suppress.
- **@ts-expect-error**: Suppresses the error on the next line, but produces an error itself if the next line has no error (the suppression is "unused").
- **Directive Comments**: Special `//` comments that control TypeScript's behavior on specific lines.

## Real World Context
During a migration, you might suppress hundreds of errors with `@ts-ignore`. Later, when you fix the underlying types, those suppressions silently remain — hiding the fact that the fix worked. `@ts-expect-error` catches this: once the error is fixed, the directive itself becomes an error, prompting you to remove it.

## Deep Dive
### @ts-ignore — The Dangerous One

```typescript
// @ts-ignore
const x: number = "hello"; // No error — suppressed silently

// Later, if you fix the type:
// @ts-ignore
const x: number = 42; // Still no error — @ts-ignore is silent
// The unnecessary suppression remains forever
```

`@ts-ignore` never complains, even when there is nothing to suppress. This makes it a maintenance hazard.

### @ts-expect-error — The Safe One

```typescript
// @ts-expect-error — intentionally wrong type for testing
const x: number = "hello"; // Suppressed

// Later, if you fix the type:
// @ts-expect-error — intentionally wrong type for testing
const x: number = 42; // Error: Unused '@ts-expect-error' directive
// TypeScript tells you the suppression is no longer needed!
```

`@ts-expect-error` is self-cleaning: when the underlying error is fixed, TypeScript alerts you to remove the directive.

### When to Use Each

| Use Case | Directive |
|----------|-----------|
| Migration — temporary suppression | `@ts-expect-error` |
| Test files — intentionally wrong types | `@ts-expect-error` |
| Working around library bugs | `@ts-expect-error` |
| Quick hack that you'll forget about | Never use `@ts-ignore` |

### Adding Explanations

Always explain WHY you are suppressing:

```typescript
// @ts-expect-error — library types are wrong, fixed in v3.0 (TODO: remove after upgrade)
const result = brokenLib.process(data);
```

## Common Pitfalls
1. **Using @ts-ignore for migration** — It creates suppressions that are never cleaned up. Always use `@ts-expect-error` instead.
2. **Suppressing without explanation** — A bare `@ts-expect-error` does not tell future developers why the suppression exists. Always add a comment.

## Best Practices
1. **Always prefer @ts-expect-error** — It is strictly better than `@ts-ignore` because it alerts you when the suppression becomes unnecessary.
2. **Add a TODO with context** — Include what needs to happen for the suppression to be removed (e.g., "remove after upgrading to v3.0").

## Summary
- `@ts-ignore` silently suppresses errors and never alerts you when they are fixed.
- `@ts-expect-error` suppresses errors but alerts you when the error no longer exists.
- Always prefer `@ts-expect-error` for self-cleaning error suppression.
- Always explain why you are suppressing with a comment.

## Code Examples

**@ts-expect-error is self-cleaning: it errors when the suppressed error no longer exists, unlike @ts-ignore which stays silent forever**

```typescript
// BAD: @ts-ignore hides the error forever
// @ts-ignore
const port: number = process.env.PORT; // silent, even after fixing

// GOOD: @ts-expect-error alerts when no longer needed
// @ts-expect-error — PORT is string | undefined, TODO: add parseInt
const port: number = process.env.PORT;

// After fixing:
const port: number = parseInt(process.env.PORT ?? "3000", 10);
// Now @ts-expect-error would produce: Unused directive
// Prompting you to clean it up!
```


## Resources

- [TypeScript: @ts-expect-error](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-9.html#-ts-expect-error-comments) — Official docs on the @ts-expect-error directive introduced in TypeScript 3.9

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*