---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-isr"
---

# Incremental Static Regeneration (ISR)

ISR allows you to update static content without rebuilding your entire site.

## How It Works

1. **Initial request**: Serves the cached page.
2. **Background revalidation**: After the cache expires, Next.js regenerates the page in the background.
3. **Subsequent requests**: Serve the updated page.

## Implementation

```typescript
// Using fetch with revalidate
async function getData() {
  const res = await fetch('https://api.example.com/data', {
    next: { revalidate: 3600 }, // Revalidate every hour
  });
  return res.json();
}

// Or using the route segment config
export const revalidate = 3600; // seconds
```

## Static vs Dynamic

| Static | Dynamic |
|--------|----------|
| Pre-rendered at build | Generated on-demand |
| Faster TTFB | Fresh data |
| Cacheable at CDN | Requires server |

ISR gives you the best of both worlds: static performance with dynamic freshness.

## Resources

- [Caching Documentation](https://nextjs.org/docs/app/building-your-application/caching) — Comprehensive caching guide

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*