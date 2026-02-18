---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-recursive-types"
---

# Recursive Types

TypeScript types can refer to themselves. This is crucial for defining structures like JSON or Trees.

```typescript
type Json = 
  | string 
  | number 
  | boolean 
  | null 
  | { [key: string]: Json } 
  | Json[];
```

## Variadic Tuple Types

TypeScript 4.0 introduced the ability to spread generic tuples.

```typescript
type Strings = [string, string];
type Numbers = [number, number];
type StrNum = [...Strings, ...Numbers]; // [string, string, number, number]

function concat<T extends unknown[], U extends unknown[]>(arr1: [...T], arr2: [...U]): [...T, ...U] {
    return [...arr1, ...arr2];
}
```

### Resources
- [TypeScript Handbook: Variadic Tuple Types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-0.html#variadic-tuple-types)

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*