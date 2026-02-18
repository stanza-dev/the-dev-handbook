---
source_course: "typescript-production"
source_lesson: "typescript-production-declaration-files"
---

# Declaration Files

Declaration files (`.d.ts`) provide type information for JavaScript code. They act as a bridge between your TS code and existing JS libraries.

## Ambient Declarations

Use `declare` to tell TypeScript about something that exists globally (like a variable injected by a script tag).

```typescript
declare const MY_GLOBAL_CONFIG: { apiUrl: string };
```

## @types Packages

Most popular libraries have types available in the DefinitelyTyped repository. You install them via `@types/library-name`.

```bash
npm install --save-dev @types/lodash
```

### Resources
- [TypeScript Handbook: Declaration Files](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html)

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*