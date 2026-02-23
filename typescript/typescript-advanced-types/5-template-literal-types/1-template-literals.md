---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-template-literals"
---

# Template Literal Types

## Introduction
Template literal types bring the power of JavaScript template strings to the type system. They let you construct string literal types by interpolating other types, generate all possible combinations from unions, and pattern-match on string shapes. Introduced in TypeScript 4.1, they are one of the features that make TypeScript's type system Turing-complete.

## Key Concepts
- **Template literal syntax**: Uses backticks and `${}` interpolation in type position: `` `hello ${World}` ``.
- **Union distribution**: When a union is interpolated, the template produces all possible combinations.
- **String literal computation**: Template literals can concatenate, transform, and pattern-match string types at compile time.

## Real World Context
Template literal types are used by CSS-in-JS libraries (generating typed class names), API route type systems (building type-safe URL patterns), event systems (deriving handler names from event names), and configuration validators (enforcing string format constraints). They bridge the gap between runtime string manipulation and compile-time type safety.

## Deep Dive

### Basic Template Literals

A template literal type constructs a new string literal type:

```typescript
type World = "world";
type Greeting = `hello ${World}`;
// type Greeting = "hello world"
```

This is a compile-time string concatenation. The result is a single string literal type.

### Interpolating Unions

When you interpolate a union, the template distributes over every member:

```typescript
type Color = "red" | "blue";
type Size = "small" | "large";

type Variant = `${Size}-${Color}`;
// "small-red" | "small-blue" | "large-red" | "large-blue"
```

TypeScript generates every possible combination. With two unions of 2 members each, you get 4 results. This is a cross product.

### Multiple Union Interpolations

You can interpolate multiple unions for larger combinations:

```typescript
type Alignment = "top" | "middle" | "bottom";
type Side = "left" | "center" | "right";

type Position = `${Alignment}-${Side}`;
// 9 combinations: "top-left" | "top-center" | ... | "bottom-right"
```

The number of resulting members is the product of all union sizes.

### Event Handler Pattern

A common real-world pattern generates handler names from event names:

```typescript
type Events = "click" | "focus" | "blur";
type HandlerName = `on${Capitalize<Events>}`;
// "onClick" | "onFocus" | "onBlur"

type Handlers = {
  [E in Events as `on${Capitalize<E>}`]: (event: E) => void;
};
// { onClick: (event: "click") => void; onFocus: ...; onBlur: ... }
```

The `Capitalize` intrinsic type uppercases the first letter. Combined with the `on` prefix, this produces standard event handler names.

### Pattern Matching with Template Literals

Template literals work in `extends` clauses for pattern matching:

```typescript
type ExtractEventName<T extends string> =
  T extends `on${infer Event}` ? Uncapitalize<Event> : never;

type A = ExtractEventName<"onClick">; // "click"
type B = ExtractEventName<"onMouseMove">; // "mouseMove"
type C = ExtractEventName<"submit">; // never (doesn't match pattern)
```

This uses `infer` within a template literal to capture the part of the string after "on", then `Uncapitalize` lowercases the first letter.

## Common Pitfalls
1. **Combinatorial explosion** — Interpolating large unions creates exponentially many members. `${A}${B}${C}` where each has 10 members produces 1000 types. TypeScript may slow down or error with very large combinations.
2. **Forgetting that only string, number, bigint, boolean, null, and undefined can be interpolated** — You cannot interpolate object types or symbols in template literals. Use `string & keyof T` to ensure you only interpolate string keys.

## Best Practices
1. **Use template literals for type-safe string APIs** — Route patterns, CSS class names, and event names all benefit from template literal types that prevent typos at compile time.
2. **Keep interpolated unions small** — Be mindful of the cross-product size. If you need large combinations, consider using `string` with a branded type instead of an exhaustive union.

## Summary
- Template literal types use backtick syntax to construct string literal types.
- Union types distribute in template positions, producing all combinations.
- Combine with `Capitalize`, `Uncapitalize`, and `infer` for string transformations.
- Pattern matching with template literals enables parsing string types.

## Code Examples

**Generating all valid CSS spacing utility class names using template literal types — ensuring only valid class combinations are used**

```typescript
// Type-safe CSS utility class builder
type Spacing = 0 | 1 | 2 | 4 | 8;
type Direction = "t" | "r" | "b" | "l" | "x" | "y";
type Prefix = "m" | "p"; // margin or padding

type SpacingClass = `${Prefix}${Direction}-${Spacing}`;
// "mt-0" | "mt-1" | ... | "py-8" (60 combinations)

function applySpacing(cls: SpacingClass): void {
  // Only valid spacing classes are accepted
}

applySpacing("mt-4");  // OK
applySpacing("px-2");  // OK
applySpacing("mz-3");  // Error: 'z' is not a valid Direction
```


## Resources

- [TypeScript Handbook: Template Literal Types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html) — Official documentation on template literal types

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*