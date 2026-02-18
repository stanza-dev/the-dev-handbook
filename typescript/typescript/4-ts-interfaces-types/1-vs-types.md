---
source_course: "typescript"
source_lesson: "typescript-interfaces-vs-types"
---

# Type Aliases

Type aliases allow you to name any type, including primitives, unions, and intersections.

```typescript
type ID = string | number;
type Point = {
  x: number;
  y: number;
};
```

# Interfaces

Interfaces are strictly for defining object shapes.

```typescript
interface Point {
  x: number;
  y: number;
}
```

## Extending

Interfaces use `extends`:
```typescript
interface Animal {
  name: string;
}
interface Bear extends Animal {
  honey: boolean;
}
```

Types use intersection `&`:
```typescript
type Animal = { name: string };
type Bear = Animal & { honey: boolean };
```

## Recommendation
Use **Interfaces** for defining object shapes (like APIs, DB models, props) because they support declaration merging and cleaner error messages. Use **Types** for unions, primitives, tuples, and complex transformations.

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*