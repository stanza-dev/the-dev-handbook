---
source_course: "typescript"
source_lesson: "typescript-narrowing-techniques"
---

# Narrowing Techniques

## Introduction
Narrowing is the process by which TypeScript refines a broad type to a more specific one based on control flow analysis. When you check a value's type with `typeof`, `instanceof`, or a property check, TypeScript understands that inside the corresponding branch, the type is narrower. This is how you safely work with union types.

## Key Concepts
- **Type Guard**: A runtime check that narrows a type within a code block.
- **`typeof` Guard**: Checks primitive types (`"string"`, `"number"`, `"boolean"`, etc.).
- **`instanceof` Guard**: Checks if a value is an instance of a class.
- **`in` Operator Narrowing**: Checks if a property exists on an object.
- **Truthiness Narrowing**: Using truthiness checks to eliminate `null`, `undefined`, and falsy values.

## Real World Context
Every time you check `if (user !== null)` before accessing `user.name`, you are narrowing. In API handlers, you narrow response types to extract data. In event handlers, you narrow event objects to access specific properties. Narrowing is something you do constantly — TypeScript just gives it a name and formal type-level support.

## Deep Dive

### typeof Narrowing

The `typeof` operator narrows primitive types:

```typescript
function formatValue(value: string | number | boolean): string {
  if (typeof value === "string") {
    return value.toUpperCase(); // value is string
  }
  if (typeof value === "number") {
    return value.toFixed(2); // value is number
  }
  return value ? "Yes" : "No"; // value is boolean
}
```

TypeScript tracks these checks through the control flow. After the first `if` eliminates `string`, the remaining type is `number | boolean`.

### instanceof Narrowing

For class instances, use `instanceof`:

```typescript
class NetworkError {
  constructor(public statusCode: number, public message: string) {}
}

class ValidationError {
  constructor(public field: string, public message: string) {}
}

function handleError(error: NetworkError | ValidationError): string {
  if (error instanceof NetworkError) {
    return `HTTP ${error.statusCode}: ${error.message}`;
  }
  return `Validation failed on ${error.field}: ${error.message}`;
}
```

After the `instanceof` check, TypeScript knows the exact class and exposes its specific properties.

### The `in` Operator

The `in` operator checks for property existence and narrows accordingly:

```typescript
type Fish = { swim: () => void; name: string };
type Bird = { fly: () => void; name: string };

function move(animal: Fish | Bird): void {
  if ("swim" in animal) {
    animal.swim(); // animal is Fish
  } else {
    animal.fly();  // animal is Bird
  }
}
```

This is particularly useful when you have object types that share some properties but differ in others.

### Truthiness Narrowing

Checking for truthiness eliminates `null`, `undefined`, `0`, `""`, and `false`:

```typescript
function greetUser(name: string | null | undefined): string {
  if (name) {
    return `Hello, ${name}!`; // name is string (non-empty)
  }
  return "Hello, guest!";
}
```

Be careful: truthiness narrowing eliminates all falsy values, including valid ones like `0` or `""`.

### Equality Narrowing

Strict equality checks also narrow types:

```typescript
function example(x: string | number, y: string | boolean): void {
  if (x === y) {
    // x and y must both be string (the only overlapping type)
    console.log(x.toUpperCase()); // x is string
    console.log(y.toUpperCase()); // y is string
  }
}
```

TypeScript deduces that if `x === y` and they share only `string` as a common type, both must be strings.

## Common Pitfalls
1. **Truthiness narrowing eliminates valid falsy values** — Checking `if (count)` will treat `0` as falsy. Use `if (count !== null && count !== undefined)` when `0` is a valid value.
2. **`typeof null` returns `"object"`** — This is a JavaScript quirk. Checking `typeof value === "object"` does not eliminate `null`. Always add a null check: `value !== null && typeof value === "object"`.

## Best Practices
1. **Prefer `typeof` for primitives and `instanceof` for classes** — Use the narrowing technique that matches the type you are checking.
2. **Use the `in` operator for discriminating object shapes** — When classes are not available (e.g., plain objects from APIs), `in` is the cleanest approach.

## Summary
- `typeof` narrows primitive types; `instanceof` narrows class instances.
- The `in` operator narrows based on property existence.
- Truthiness checks eliminate `null` and `undefined` but also other falsy values.
- TypeScript's control flow analysis tracks narrowing through if/else, switch, and ternary expressions.

## Code Examples

**The 'in' operator checks for a property unique to one union member, letting TypeScript narrow to the correct shape**

```typescript
// Combining multiple narrowing techniques
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rectangle"; width: number; height: number };

function getArea(shape: Shape): number {
  if ("radius" in shape) {
    // Narrowed to circle
    return Math.PI * shape.radius ** 2;
  }
  // Narrowed to rectangle
  return shape.width * shape.height;
}

console.log(getArea({ kind: "circle", radius: 5 }));              // 78.54
console.log(getArea({ kind: "rectangle", width: 10, height: 3 })); // 30
```


## Resources

- [TypeScript Handbook: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html) — Complete guide to typeof, instanceof, in, and other narrowing techniques

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*