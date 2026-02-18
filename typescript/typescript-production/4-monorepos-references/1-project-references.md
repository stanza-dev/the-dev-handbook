---
source_course: "typescript-production"
source_lesson: "typescript-production-project-references"
---

# Project References

Project references allow you to structure your TypeScript programs into smaller pieces.

## The `composite` Flag

To use project references, referenced projects must have the `composite` setting enabled in their `tsconfig.json`. This ensures TypeScript can quickly determine if a project needs rebuilding.

```json
// libs/core/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true
  }
}
```

## Referencing

In your main project:
```json
{
  "references": [
    { "path": "../libs/core" }
  ]
}
```

### Resources
- [TypeScript Handbook: Project References](https://www.typescriptlang.org/docs/handbook/project-references.html)

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*