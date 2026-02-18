---
source_course: "nextjs"
source_lesson: "nextjs-use-cache-directive"
---

# The 'use cache' Directive

The `'use cache'` directive is used to cache the result of a function or component. This is part of Next.js's advanced caching mechanisms to improve performance.

When you mark a function or component with `'use cache'`, its return value is stored and reused for subsequent requests, reducing server load and computation time.

```tsx
'use cache'

export async function CachedData() {
  const data = await heavyComputation()
  return <div>{data}</div>
}
```

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*