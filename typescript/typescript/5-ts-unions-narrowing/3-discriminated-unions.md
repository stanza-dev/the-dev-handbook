---
source_course: "typescript"
source_lesson: "typescript-discriminated-unions"
---

# Discriminated Unions

## Introduction
Discriminated unions (also called tagged unions) are a pattern where each member of a union has a common property with a unique literal value. This "tag" property lets TypeScript narrow the type automatically in switch statements and if checks, making it impossible to miss a case.

## Key Concepts
- **Discriminant Property**: A shared property across union members, each with a unique literal type (the "tag").
- **Exhaustiveness Checking**: Using the `never` type to ensure every variant of a union is handled.
- **Switch Narrowing**: TypeScript narrows the union type within each `case` of a switch statement on the discriminant.

## Real World Context
State machines, Redux actions, API responses, and event systems all use discriminated unions. A notification system might have `{ type: "email" }`, `{ type: "sms" }`, and `{ type: "push" }` — each with different required fields. The discriminant `type` ensures you handle each notification correctly.

## Deep Dive

### The Discriminant Pattern

Every member of the union shares a property with a unique literal value:

```typescript
type LoadingState = {
  status: "loading";
};

type SuccessState = {
  status: "success";
  data: string[];
};

type ErrorState = {
  status: "error";
  message: string;
  retryable: boolean;
};

type RequestState = LoadingState | SuccessState | ErrorState;
```

The `status` property is the discriminant. Its value is a unique string literal in each variant.

### Switch Narrowing

TypeScript narrows the type in each case:

```typescript
function renderState(state: RequestState): string {
  switch (state.status) {
    case "loading":
      return "Loading...";
    case "success":
      return `Found ${state.data.length} items`; // data is available
    case "error":
      return `Error: ${state.message}`; // message is available
  }
}
```

Inside `case "success"`, TypeScript knows `state` is `SuccessState`, so `state.data` is accessible.

### Exhaustiveness with `never`

You can ensure every variant is handled by assigning the narrowed value to `never` in the default case:

```typescript
function handleState(state: RequestState): string {
  switch (state.status) {
    case "loading":
      return "Loading...";
    case "success":
      return `Got ${state.data.length} items`;
    case "error":
      return `Error: ${state.message}`;
    default: {
      const exhaustiveCheck: never = state;
      return exhaustiveCheck; // This line is unreachable if all cases are handled
    }
  }
}
```

If you later add a new variant to `RequestState` (like `{ status: "cancelled" }`) but forget to add a `case` for it, the assignment to `never` will produce a compile error, alerting you immediately.

### Real-World Example: Action Handling

Discriminated unions are the foundation of action/event patterns:

```typescript
type Action =
  | { type: "ADD_ITEM"; payload: { name: string; price: number } }
  | { type: "REMOVE_ITEM"; payload: { id: string } }
  | { type: "CLEAR_CART" };

function cartReducer(state: CartState, action: Action): CartState {
  switch (action.type) {
    case "ADD_ITEM":
      return { ...state, items: [...state.items, action.payload] };
    case "REMOVE_ITEM":
      return { ...state, items: state.items.filter(i => i.id !== action.payload.id) };
    case "CLEAR_CART":
      return { ...state, items: [] };
  }
}
```

Each action has different payload shapes, and TypeScript narrows to the correct one in each case.

## Common Pitfalls
1. **Forgetting to add a case for new variants** — Without exhaustiveness checking, adding a new union member silently falls through. Always include a `default` case with `never` assignment.
2. **Using a non-literal discriminant** — The discriminant must be a literal type (`"loading"`, not `string`). If the property is typed as `string`, narrowing will not work.

## Best Practices
1. **Always implement exhaustiveness checking** — The `never` pattern in the default case catches missing cases at compile time.
2. **Use consistent discriminant names** — Pick a convention (`type`, `kind`, `status`) and use it consistently across your codebase.

## Summary
- Discriminated unions use a shared literal property (tag) to distinguish variants.
- Switch statements on the tag automatically narrow the type.
- Exhaustiveness checking with `never` ensures all variants are handled.
- This pattern is foundational for state machines, reducers, and event systems.

## Code Examples

**The never type in the default case ensures every Shape variant has a corresponding case — adding a new shape without a case causes a compile error**

```typescript
// Exhaustiveness checking catches missing cases at compile time
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "triangle"; base: number; height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default: {
      const _exhaustive: never = shape;
      return _exhaustive;
    }
  }
}
```


## Resources

- [TypeScript Handbook: Narrowing (Discriminated Unions)](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions) — Official guide to discriminated unions and exhaustiveness checking

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*