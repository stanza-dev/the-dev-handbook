---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-pick-omit-partial"
---

# Built-in Utility Types

TypeScript provides several utility types to facilitate common type transformations.

## Partial<Type>
Constructs a type with all properties of `Type` set to optional.

```typescript
interface Todo { title: string; description: string; }
function updateTodo(todo: Todo, fields: Partial<Todo>) { ... }
```

## Pick<Type, Keys>
Constructs a type by picking the set of properties `Keys` from `Type`.

```typescript
type TodoPreview = Pick<Todo, "title">;
```

## Omit<Type, Keys>
Constructs a type by picking all properties from `Type` and then removing `Keys`.

```typescript
type TodoPreview = Omit<Todo, "description">;
```

## Record<Keys, Type>
Constructs an object type whose property keys are `Keys` and whose property values are `Type`.

```typescript
const nav: Record<string, string> = { about: "/about", contact: "/contact" };
```

### Resources
- [TypeScript Handbook: Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*