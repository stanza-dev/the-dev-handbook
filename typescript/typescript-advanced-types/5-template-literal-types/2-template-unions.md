---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-template-unions"
---

# Union Distribution in Template Literals

## Introduction
Template literal types become especially powerful when combined with union types. TypeScript automatically generates all possible string combinations when unions appear in template positions. This lesson explores the mechanics of this distribution, how to control it, and practical patterns for generating exhaustive type-safe string sets.

## Key Concepts
- **Cross-product distribution**: Each union in a template position contributes its members independently, producing every combination.
- **Size calculation**: The number of resulting members equals the product of all interpolated union sizes.
- **Combining with mapped types**: Template literal distribution works inside mapped types for generating entire object type surfaces.

## Real World Context
This pattern is used by Tailwind CSS type definitions to generate all valid class names, by internationalization libraries to type locale keys, by database query builders to type column expressions, and by testing frameworks to generate method names like `toBeGreaterThan`, `toBeLessThan`, etc.

## Deep Dive

### How Distribution Works

When a union appears in a template position, TypeScript evaluates the template for every member:

```typescript
type Fruit = "apple" | "banana";
type Action = "eat" | "buy" | "sell";

type FruitAction = `${Action}_${Fruit}`;
// "eat_apple" | "eat_banana" | "buy_apple" | "buy_banana" | "sell_apple" | "sell_banana"
```

This produces 2 x 3 = 6 members. Each `Action` member is combined with each `Fruit` member.

### Multiple Positions

Multiple union positions multiply together:

```typescript
type A = "a" | "b";        // 2 members
type B = "x" | "y" | "z";  // 3 members
type C = "1" | "2";        // 2 members

type All = `${A}-${B}-${C}`; // 2 * 3 * 2 = 12 members
// "a-x-1" | "a-x-2" | "a-y-1" | ... | "b-z-2"
```

The total is always the product of all union sizes in the template.

### Generating Object Types from Combinations

Combine template literal distribution with mapped types to generate object shapes:

```typescript
type Axis = "x" | "y" | "z";
type Bound = "min" | "max";

type BoundingBox = {
  [K in `${Bound}${Capitalize<Axis>}`]: number;
};
// { minX: number; minY: number; minZ: number; maxX: number; maxY: number; maxZ: number }
```

The mapped type iterates over the 6 template literal combinations and creates a property for each.

### Practical: Type-Safe Route Parameters

Template literals can enforce URL parameter patterns:

```typescript
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type Resource = "users" | "posts" | "comments";

type Endpoint = `${Uppercase<HTTPMethod>} /api/${Resource}`;
// "GET /api/users" | "GET /api/posts" | ... | "DELETE /api/comments"

type EndpointWithId = `${Uppercase<HTTPMethod>} /api/${Resource}/:id`;
// "GET /api/users/:id" | ... (12 combinations)

function registerRoute(endpoint: Endpoint | EndpointWithId): void {
  // Only valid combinations accepted
}

registerRoute("GET /api/users");       // OK
registerRoute("DELETE /api/posts/:id"); // OK
registerRoute("PATCH /api/users");      // Error: PATCH not in HTTPMethod
```

This enforces that only valid method + resource combinations are used as routes.

### Size Limits and Performance

TypeScript limits the number of members in a union created by template literal distribution. As of TypeScript 5.x, the practical limit is around 100,000 members before performance degrades significantly:

```typescript
// This is fine (small unions):
type Small = `${"a" | "b"}-${"x" | "y"}`; // 4 members

// This is risky (large cross product):
// type Huge = `${LargeUnion1}-${LargeUnion2}-${LargeUnion3}`;
// Could produce millions of members — avoid this!
```

When you hit these limits, consider using a branded `string` type with runtime validation instead.

## Common Pitfalls
1. **Accidentally creating huge unions** — Each interpolated union multiplies the result size. Three unions of 10 members each create 1,000 combinations. Monitor the sizes.
2. **Expecting runtime string validation** — Template literal types are compile-time only. They do not validate strings at runtime. Pair them with runtime checks when accepting user input.

## Best Practices
1. **Use small unions for combinatorial generation** — Template literal distribution works best with small, focused unions (2-10 members each).
2. **Leverage `Uppercase`/`Lowercase`/`Capitalize` for normalization** — These intrinsic types let you standardize casing in template positions without manually listing variants.

## Summary
- Template literal types distribute over unions, producing the cross product of all combinations.
- The result size is the product of all interpolated union sizes.
- Mapped types over template literal unions generate entire object type surfaces.
- Keep union sizes small to avoid performance issues from combinatorial explosion.

## Code Examples

**Using template literal distribution to generate all valid CSS grid column classes with responsive breakpoint prefixes**

```typescript
// Building a typed CSS grid system
type Column = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12;
type Breakpoint = "sm" | "md" | "lg" | "xl";

// Simple column class: "col-1" through "col-12"
type ColClass = `col-${Column}`;

// Responsive column class: "sm:col-1" through "xl:col-12"
type ResponsiveColClass = `${Breakpoint}:col-${Column}`;

// All valid column classes
type AnyColClass = ColClass | ResponsiveColClass;
// 12 + (4 * 12) = 60 members

function setGrid(cls: AnyColClass): void { /* ... */ }
setGrid("col-6");       // OK
setGrid("md:col-4");    // OK
setGrid("col-13");      // Error: 13 is not in Column
setGrid("xs:col-1");    // Error: 'xs' is not in Breakpoint
```


## Resources

- [TypeScript Handbook: Template Literal Types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html) — Official documentation covering template literal type distribution

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*