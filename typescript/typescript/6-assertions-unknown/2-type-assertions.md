---
source_course: "typescript"
source_lesson: "typescript-type-assertions"
---

# Type Assertions

Sometimes you know more about a value's type than TypeScript does. You can use the `as` keyword to tell the compiler "trust me".

```typescript
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

TypeScript only allows assertions that seem plausible (e.g., converting a more specific type to a less specific one or vice versa). For impossible casts, use `unknown` as an intermediate step:

```typescript
const x = "hello" as unknown as number;
```

**Note**: Avoid using `<Type>` syntax in `.tsx` files as it conflicts with JSX.

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*