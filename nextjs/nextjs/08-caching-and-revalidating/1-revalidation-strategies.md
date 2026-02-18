---
source_course: "nextjs"
source_lesson: "nextjs-revalidation-strategies"
---

# Caching and Revalidating

Next.js provides two main ways to revalidate (refresh) data.

## 1. Time-based Revalidation

Automatically revalidate data after a set period of time. Useful for data specifically that doesn't change often.

```tsx
fetch('https://...', { next: { revalidate: 3600 } })
```

## 2. On-demand Revalidation

Manually revalidate data when an event occurs (e.g., form submission). Useful for ensuring data is fresh immediately after an update.

*   `revalidatePath('/blog')`: Revalidates a specific route.
*   `revalidateTag('posts')`: Revalidates any cached data tagged with 'posts'.

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*