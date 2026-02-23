---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-functional-composition"
---

# Function Composition & Currying

## Introduction
Functional composition lets you build complex transformations from small, reusable functions. TypeScript's type system can track types through composition chains, giving you both the flexibility of functional programming and the safety of static types.

## Key Concepts
- **Pipe**: Passes a value through a sequence of functions left-to-right: `pipe(x, f, g)` equals `g(f(x))`.
- **Compose**: Passes a value through functions right-to-left: `compose(f, g)(x)` equals `f(g(x))`.
- **Currying**: Transforming a multi-argument function into a chain of single-argument functions: `f(a, b)` becomes `f(a)(b)`.
- **Partial Application**: Fixing some arguments of a function to produce a new function with fewer parameters.

## Real World Context
Libraries like Ramda and fp-ts rely heavily on composition and currying. Data pipelines, middleware stacks, and validation chains all use sequential function application. TypeScript's generics make these patterns fully type-safe.

## Deep Dive
### The Pipe Function

A simple two-function pipe:

```typescript
function pipe<A, B, C>(value: A, fn1: (a: A) => B, fn2: (b: B) => C): C {
    return fn2(fn1(value));
}

const result = pipe(
    "  Hello World  ",
    (s) => s.trim(),
    (s) => s.toLowerCase()
);
// result: "hello world"
```

TypeScript infers each intermediate type. For more functions, use overloads or a variadic approach:

```typescript
function pipe<A>(value: A): A;
function pipe<A, B>(value: A, fn1: (a: A) => B): B;
function pipe<A, B, C>(value: A, fn1: (a: A) => B, fn2: (b: B) => C): C;
function pipe<A, B, C, D>(value: A, fn1: (a: A) => B, fn2: (b: B) => C, fn3: (c: C) => D): D;
function pipe(value: unknown, ...fns: Function[]): unknown {
    return fns.reduce((acc, fn) => fn(acc), value);
}
```

### Currying

Currying transforms a multi-argument function into nested single-argument functions:

```typescript
function curry<A, B, C>(fn: (a: A, b: B) => C): (a: A) => (b: B) => C {
    return (a) => (b) => fn(a, b);
}

const add = curry((a: number, b: number) => a + b);
const add5 = add(5);  // (b: number) => number
add5(3); // 8
```

Curried functions are ideal for creating reusable, partially applied utilities.

### Practical Example: Validation Pipeline

```typescript
type Validator<T> = (value: T) => string | null;

function combineValidators<T>(...validators: Validator<T>[]): Validator<T> {
    return (value) => {
        for (const validate of validators) {
            const error = validate(value);
            if (error) return error;
        }
        return null;
    };
}

const isNonEmpty: Validator<string> = (s) => s.length === 0 ? "Required" : null;
const isEmail: Validator<string> = (s) => s.includes("@") ? null : "Invalid email";

const validateEmail = combineValidators(isNonEmpty, isEmail);
validateEmail("");              // "Required"
validateEmail("bad");           // "Invalid email"
validateEmail("alice@test.com"); // null (valid)
```

## Common Pitfalls
1. **Losing type inference in long chains** — TypeScript struggles to infer types through more than ~4 composed functions. Use explicit type annotations or overloads to help.
2. **Over-currying** — Currying every function makes code harder to read. Curry only functions that benefit from partial application.

## Best Practices
1. **Prefer pipe over compose** — Left-to-right reading order matches how most developers think about data flow.
2. **Use overloads for pipe** — Provide typed overloads for 2, 3, 4, and 5-function pipes to maintain full type inference.

## Summary
- Pipe passes a value through functions left-to-right; compose goes right-to-left.
- Currying creates reusable partially-applied functions.
- TypeScript's generics and overloads keep composition chains fully type-safe.
- Prefer pipe over compose for readability.

## Code Examples

**A curried filter factory — create reusable filter functions by partially applying the predicate**

```typescript
// Curried filter factory
const filterBy = <T>(predicate: (item: T) => boolean) =>
    (items: T[]): T[] => items.filter(predicate);

const isAdult = filterBy<{ age: number }>(user => user.age >= 18);

const users = [
    { name: "Alice", age: 25 },
    { name: "Bob", age: 16 },
    { name: "Carol", age: 30 }
];

const adults = isAdult(users);
// [{ name: "Alice", age: 25 }, { name: "Carol", age: 30 }]
```


## Resources

- [fp-ts Documentation](https://gcanti.github.io/fp-ts/) — A popular TypeScript library for typed functional programming with pipe, compose, and more

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*