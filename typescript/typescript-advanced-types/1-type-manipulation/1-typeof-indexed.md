---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-keyof-typeof-indexed"
---

# The `keyof` Operator

The `keyof` operator takes an object type and produces a string or numeric literal union of its keys.

```typescript
type Point = { x: number; y: number };
type P = keyof Point; // "x" | "y"
```

# The `typeof` Operator

TypeScript allows you to use `typeof` in a type context to refer to the *type* of a variable.

```typescript
const config = { width: 100, height: 200 };
type Config = typeof config; // { width: number; height: number }
```

# Indexed Access Types

You can use `Type[Key]` to look up a specific property's type.

```typescript
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"]; // number
```

### Resources
- [TypeScript Handbook: keyof Type Operator](https://www.typescriptlang.org/docs/handbook/2/keyof-types.html)

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*