---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-result-types"
---

# Result Pattern

Instead of throwing errors (which are not type-safe in TS signatures), many developers prefer returning a `Result` object.

```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

function parseUser(json: string): Result<User> {
  try {
    return { success: true, data: JSON.parse(json) };
  } catch (e) {
    return { success: false, error: e as Error };
  }
}
```

This forces the caller to check `success` before accessing `data` (via Discriminated Unions).

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*