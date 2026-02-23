---
source_course: "typescript"
source_lesson: "typescript-unions"
---

# Union Types

## Introduction
Union types are one of TypeScript's most powerful features. They allow a value to be one of several types, modeling real-world scenarios where data can come in different shapes. Combined with narrowing, unions enable you to write code that is both flexible and type-safe.

## Key Concepts
- **Union Type**: A type formed by combining two or more types with the `|` operator, meaning the value can be any one of them.
- **Common Members**: On a union, you can only access properties and methods that exist on every member of the union.
- **Narrowing**: The process of refining a union type to a more specific type using type guards.

## Real World Context
APIs often return different shapes for success and error cases. Form inputs can be strings or numbers depending on the field. Database queries might return a record or `null`. Union types model all of these scenarios naturally, and narrowing ensures you handle each case correctly.

## Deep Dive

### Defining Union Types

A union type uses the pipe `|` operator:

```typescript
type StringOrNumber = string | number;
type Theme = "light" | "dark" | "system";
type ApiResult = SuccessResponse | ErrorResponse | null;
```

You can use unions anywhere a type is expected — variables, parameters, return types, and generics.

### Working with Unions

When a value has a union type, TypeScript only allows operations valid for all members:

```typescript
function processValue(value: string | number): string {
  // value.toUpperCase(); // Error: 'toUpperCase' does not exist on 'number'
  // value.toFixed();     // Error: 'toFixed' does not exist on 'string'
  return String(value);   // String() works for both — OK
}
```

To use type-specific methods, you must narrow the type first:

```typescript
function processValue(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase(); // TypeScript knows value is string here
  }
  return value.toFixed(2); // TypeScript knows value is number here
}
```

The `typeof` check narrows the union, giving you full access to type-specific methods in each branch.

### Literal Union Types

Literal types combined with unions create precise, self-documenting types:

```typescript
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE" | "PATCH";
type LogLevel = "debug" | "info" | "warn" | "error";
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;

function logMessage(level: LogLevel, message: string): void {
  const timestamp = new Date().toISOString();
  console.log(`[${timestamp}] ${level.toUpperCase()}: ${message}`);
}

logMessage("info", "Server started");
// logMessage("verbose", "..."); // Error: '"verbose"' is not assignable to type 'LogLevel'
```

Literal unions are often preferred over enums for simple string sets because they require no extra runtime code.

### Unions with `null` and `undefined`

With `strictNullChecks` enabled, `null` and `undefined` are not assignable to other types unless you explicitly include them:

```typescript
function findUser(id: string): User | null {
  const user = database.get(id);
  return user ?? null;
}

const user = findUser("u123");
// user.name; // Error: 'user' is possibly 'null'
if (user) {
  console.log(user.name); // OK — narrowed to User
}
```

This forces you to handle the null case, preventing null reference errors.

## Common Pitfalls
1. **Forgetting to narrow before accessing type-specific members** — On a union, only shared members are available. Use `typeof`, `instanceof`, or property checks to narrow.
2. **Overly wide unions** — A union like `string | number | boolean | null | undefined | object` is almost as useless as `any`. Keep unions focused on the actual variants.

## Best Practices
1. **Prefer literal unions for known string sets** — `"success" | "error" | "pending"` is clearer and lighter than an enum.
2. **Always handle every variant** — When consuming a union, use exhaustive checks (covered in the Discriminated Unions lesson) to ensure no case is missed.

## Summary
- Union types allow a value to be one of several types using the `|` operator.
- You can only access members common to all union variants without narrowing.
- Literal unions create precise types without runtime overhead.
- Always narrow a union before using type-specific operations.

## Code Examples

**A function that accepts either a number or string for padding, narrowing the type with typeof to handle each case**

```typescript
// Union types for flexible, type-safe function parameters
type Padding = number | string;

function applyPadding(value: string, padding: Padding): string {
  if (typeof padding === "number") {
    return " ".repeat(padding) + value;
  }
  return padding + value;
}

console.log(applyPadding("Hello", 4));     // "    Hello"
console.log(applyPadding("Hello", ">> ")); // ">> Hello"
```


## Resources

- [TypeScript Handbook: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html) — Official guide to union types and type narrowing techniques

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*