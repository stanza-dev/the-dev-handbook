---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-abstract-classes-mixins"
---

# Abstract Classes

Abstract classes are base classes that cannot be instantiated directly. They may contain implementation details for their members.

```typescript
abstract class Animal {
  abstract makeSound(): void;
  move(): void {
    console.log("roaming the earth...");
  }
}
```

## Mixins

TypeScript models mixins as functions that accept a class constructor and return a new class extending it.

```typescript
type Constructor = new (...args: any[]) => {};

function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}
```

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*