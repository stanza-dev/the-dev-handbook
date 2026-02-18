---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-template-literals"
---

# Template Literal Types

Template literal types share the same syntax as JavaScript template strings, but are used in type positions.

```typescript
type World = "world";
type Greeting = `hello ${World}`;
// type Greeting = "hello world"
```

## String Unions

They distribute over unions, allowing you to generate many combinations.

```typescript
type Color = "red" | "blue";
type Quantity = "light" | "dark";
type Palette = `${Quantity}-${Color}`;
// "light-red" | "light-blue" | "dark-red" | "dark-blue"
```

## Intrinsic String Manipulation

TypeScript provides helpers: `Uppercase<T>`, `Lowercase<T>`, `Capitalize<T>`, `Uncapitalize<T>`.

```typescript
type Shouty = Uppercase<"hello">; // "HELLO"
```

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*