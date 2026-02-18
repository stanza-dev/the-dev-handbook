---
source_course: "nextjs"
source_lesson: "nextjs-error-boundaries"
---

# Error Handling

The `error.js` file convention allows you to gracefully handle runtime errors in nested routes.

## `error.js`

*   Must be a **Client Component** (`'use client'`).
*   Wraps a route segment and its children in a React Error Boundary.
*   Receives `error` and `reset` props.

```tsx
'use client'

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}
```

## `global-error.js`

To handle errors in the root layout, use `global-error.js`. It replaces the root layout and must define its own `html` and `body` tags.

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*