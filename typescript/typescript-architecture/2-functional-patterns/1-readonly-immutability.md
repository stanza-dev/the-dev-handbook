---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-readonly-immutability"
---

# Immutability

TypeScript provides tools to enforce immutability at compile time.

## Readonly Arrays and Tuples

```typescript
const list: readonly number[] = [1, 2, 3];
// list.push(4); // Error
```

## Const Assertions

The most powerful tool is `as const`.

```typescript
const config = {
  endpoint: "https://api.example.com",
  retries: 3
} as const;

// Type is:
// {
//   readonly endpoint: "https://api.example.com";
//   readonly retries: 3;
// }
```

It makes everything `readonly` and narrows literals to their specific values.

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*