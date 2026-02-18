---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-nominal-typing"
---

# Nominal vs Structural Typing

TypeScript is structurally typed: if it looks like a duck, it's a duck. `interface A { x: number }` is compatible with `interface B { x: number }`.

Sometimes, you want **Nominal Typing** (distinguishing types by name) to prevent mixing up values, like `UserId` and `OrderId`.

## Branded Types

We can simulate this using "branding" (intersection with a unique tag).

```typescript
type Brand<K, T> = K & { __brand: T };

type USD = Brand<number, "USD">;
type EUR = Brand<number, "EUR">;

function euroToUsd(euro: EUR): USD {
  return (euro * 1.1) as USD;
}

const myEur = 10 as EUR;
const myUsd = euroToUsd(myEur);
// euroToUsd(10); // Error: number is not EUR
```

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*