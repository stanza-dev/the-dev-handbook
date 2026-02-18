---
source_course: "typescript-production"
source_lesson: "typescript-production-jsdoc-ts-check"
---

# JSDoc Support

TypeScript can check JavaScript files using JSDoc comments.

Add `// @ts-check` to the top of a `.js` file to enable type checking.

```javascript
// @ts-check

/**
 * @param {string} name
 * @returns {string}
 */
function greet(name) {
  return "Hello " + name;
}
```

## The `@satisfies` Tag

TypeScript 5.0 introduced `@satisfies` to check if a value matches a type without widening it (similar to the `satisfies` operator in TS).

```javascript
/** @satisfies {CompilerOptions} */
let config = { outDir: "./dist" };
```

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*