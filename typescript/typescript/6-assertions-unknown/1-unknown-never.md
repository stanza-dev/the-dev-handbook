---
source_course: "typescript"
source_lesson: "typescript-unknown-never"
---

# The `unknown` Type

`unknown` is the type-safe counterpart of `any`. You can assign anything to `unknown`, but you cannot do anything with it until you narrow it.

```typescript
let value: unknown;
value = true;

// let x: boolean = value; // Error
if (typeof value === "boolean") {
  let x: boolean = value; // OK
}
```

## The `never` Type

`never` represents values that never occur (like a function that always throws an error). It's useful in exhaustive checks.

```typescript
function fail(msg: string): never {
  throw new Error(msg);
}
```

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*