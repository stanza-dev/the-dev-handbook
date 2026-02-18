---
source_course: "typescript"
source_lesson: "typescript-primitives-arrays-any"
---

# Primitives

TypeScript supports the same primitives as JavaScript: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, and `bigint`.

```typescript
const username: string = "Alice";
const count: number = 42;
const isActive: boolean = true;
```

## Arrays

You can define arrays in two ways:

```typescript
let list: number[] = [1, 2, 3];
let genericList: Array<number> = [1, 2, 3]; // Generic syntax
```

## The `any` Type

The `any` type opts out of type checking. It's useful when migrating JS code but should be avoided in new code because it defeats the purpose of TypeScript.

```typescript
let obj: any = { x: 0 };
obj.foo(); // No error at compile time, but might crash at runtime!
```

## Type Inference

Often, you don't need to specify types. TypeScript infers them for you:

```typescript
let x = 20; // TypeScript knows x is a number
```

### Resources
- [TypeScript Handbook: Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*