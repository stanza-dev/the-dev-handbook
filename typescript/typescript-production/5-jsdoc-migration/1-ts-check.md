---
source_course: "typescript-production"
source_lesson: "typescript-production-jsdoc-ts-check"
---

# JSDoc Type Checking with @ts-check

## Introduction
You do not need to rename all your `.js` files to `.ts` to start using TypeScript. By adding `// @ts-check` at the top of a JavaScript file, TypeScript will type-check it using JSDoc annotations. This is the gentlest migration path: zero build changes, zero file renames, incremental type safety.

## Key Concepts
- **@ts-check**: A comment directive that enables TypeScript checking in a `.js` file.
- **JSDoc Type Annotations**: TypeScript-aware JSDoc comments that provide type information in JavaScript files.
- **checkJs**: A tsconfig flag that enables type checking for ALL `.js` files (equivalent to adding `@ts-check` everywhere).

## Real World Context
A large legacy JavaScript codebase with 500 files cannot be converted to TypeScript overnight. Adding `@ts-check` to one file at a time lets you incrementally catch bugs without disrupting the existing build pipeline or deployment process.

## Deep Dive
### Enabling @ts-check

Add a single comment to the top of any `.js` file:

```javascript
// @ts-check

/**
 * @param {string} name
 * @returns {string}
 */
function greet(name) {
    return "Hello, " + name;
}

greet(42); // Error: Argument of type 'number' is not assignable to parameter of type 'string'
```

TypeScript reads the JSDoc annotations and checks the code without any build step changes.

### Common JSDoc Type Annotations

```javascript
// @ts-check

/** @type {string} */
let username = "Alice";

/** @type {number[]} */
let scores = [100, 85, 92];

/** @type {{ name: string, age: number }} */
let user = { name: "Bob", age: 30 };

/**
 * @param {string} text
 * @param {number} [times=1] — optional parameter with default
 * @returns {string}
 */
function repeat(text, times = 1) {
    return text.repeat(times);
}

/** @typedef {{ id: string, email: string }} User */

/** @type {User} */
let currentUser = { id: "1", email: "alice@test.com" };
```

### The @satisfies Tag (TypeScript 5.0+)

Similar to the `satisfies` operator in `.ts` files:

```javascript
// @ts-check

/**
 * @satisfies {Record<string, number>}
 */
const sizes = {
    small: 12,
    medium: 16,
    large: 20,
};

sizes.small; // Type is 'number', not 'string | number'
```

### Project-Wide Checking

Instead of adding `@ts-check` to each file, enable it globally:

```json
{
    "compilerOptions": {
        "allowJs": true,
        "checkJs": true,
        "noEmit": true
    },
    "include": ["src/**/*"]
}
```

## Common Pitfalls
1. **Incomplete JSDoc annotations** — If you skip annotations on function parameters, they default to `any`. TypeScript only checks what you annotate.
2. **JSDoc syntax differences** — JSDoc uses `{Type}` in curly braces, not `: Type` after the name. Getting the syntax wrong silently produces `any`.

## Best Practices
1. **Start with @ts-check on critical files** — Add it to files that handle money, authentication, or data validation first — where bugs are most expensive.
2. **Use @typedef for shared types** — Define complex types once with `@typedef` and reference them across files.

## Summary
- `// @ts-check` enables TypeScript checking in individual `.js` files.
- JSDoc annotations use `{Type}` syntax for parameters, return values, and variables.
- `checkJs: true` in tsconfig enables checking for all JavaScript files at once.
- This is the lowest-friction path to adding type safety to a JavaScript project.

## Code Examples

**Full JSDoc type checking in a JavaScript file — typedef defines a reusable type, parameters are checked, and errors are caught**

```javascript
// @ts-check

/**
 * @typedef {Object} Product
 * @property {string} id
 * @property {string} name
 * @property {number} price
 * @property {string[]} tags
 */

/**
 * Calculate total price with tax
 * @param {Product[]} products
 * @param {number} taxRate
 * @returns {number}
 */
function calculateTotal(products, taxRate) {
    const subtotal = products.reduce((sum, p) => sum + p.price, 0);
    return subtotal * (1 + taxRate);
}

// TypeScript catches errors even in .js files!
// calculateTotal("not an array", 0.1); // Error
```


## Resources

- [TypeScript: JS Projects with JSDoc](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html) — Official reference for all JSDoc types that TypeScript supports

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*