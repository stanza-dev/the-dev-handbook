---
source_course: "typescript"
source_lesson: "typescript-generics-basics"
---

# Generics

Imagine you want a function that returns whatever you pass into it. Without generics, you might use `any`, but then you lose type safety.

Generics allow you to create a "type variable" that captures the type provided by the user.

```typescript
function identity<T>(arg: T): T {
  return arg;
}

// Usage
let output = identity<string>("myString");
// T becomes 'string', so the return type is 'string'
```

## Generic Interfaces

```typescript
interface Box<Type> {
  contents: Type;
}

let box: Box<string> = { contents: "hello" };
```

### Resources
- [TypeScript Handbook: Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*