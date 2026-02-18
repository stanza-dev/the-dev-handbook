---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-decorators-intro"
---

# Decorators

Decorators provide a way to add both annotations and a meta-programming syntax for class declarations and members. They are a stage 3 proposal for JavaScript and widely used in frameworks like NestJS.

To use them, enable `experimentalDecorators` in `tsconfig.json`.

```typescript
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class BugReport {
  type = "report";
}
```

They can be attached to classes, methods, accessors, properties, and parameters.

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*