---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-conditional-types-basics"
---

# Conditional Types

Conditional types allow you to choose types based on other types. They take the form:

```typescript
SomeType extends OtherType ? TrueType : FalseType;
```

Example:

```typescript
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
```

## The `infer` Keyword

Within the `extends` clause of a conditional type, you can use `infer` to declare a type variable to be inferred.

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

Here, we check if `T` is a function. If it is, we infer its return type as `R` and return `R`. Otherwise, we return `any`.

### Resources
- [TypeScript Handbook: Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*