---
source_course: "typescript"
source_lesson: "typescript-functions-objects"
---

# Functions

You can add types to parameters and return values.

```typescript
function add(a: number, b: number): number {
  return a + b;
}
```

If you don't provide a return type, TypeScript will infer it based on the `return` statements.

## Object Types

To define an object's shape:

```typescript
function printCoord(pt: { x: number; y: number }) {
  console.log("x: " + pt.x);
  console.log("y: " + pt.y);
}
```

## Optional Properties

Use `?` to mark a property as optional. If accessed, it may be `undefined`.

```typescript
function printName(obj: { first: string; last?: string }) {
  // obj.last might be undefined here
}
```

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*