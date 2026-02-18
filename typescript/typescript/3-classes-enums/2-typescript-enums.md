---
source_course: "typescript"
source_lesson: "typescript-enums"
---

# Enums

Enums allow you to define a set of named constants. TypeScript provides both numeric and string-based enums.

## Numeric Enums

By default, enums are zero-indexed.

```typescript
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right  // 3
}
```

## String Enums

String enums are more readable when debugging.

```typescript
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
```

Use enums when you have a closed set of related values (like user roles, status codes, or directions).

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*