---
source_course: "typescript"
source_lesson: "typescript-unions"
---

# Union Types

A union type is a value that can be one of several types. It's like saying "this is a string OR a number".

```typescript
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
```

If you have a union, you can only access members valid for *all* types in the union. To use type-specific methods (like `toUpperCase()` for strings), you need **Narrowing**.

## Narrowing with Type Guards

TypeScript is smart enough to look at your control flow checks (if/else, switch).

```typescript
function printId(id: number | string) {
  if (typeof id === "string") {
    // In this block, TS knows id is a string
    console.log(id.toUpperCase());
  } else {
    // Here, id must be a number
    console.log(id);
  }
}
```

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*