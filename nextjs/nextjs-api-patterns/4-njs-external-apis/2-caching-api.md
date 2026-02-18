---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-caching-api"
---

# Caching API Responses

## Introduction

Caching is the key to fast APIs. By storing responses for repeated requests, you reduce database load, speed up response times, and save money on serverless function invocations.

## Key Concepts

**Next.js caching layers**:

- **Route Segment Config**: Export `revalidate` to set cache duration
- **Fetch caching**: Per-request caching via `next: { revalidate }`
- **HTTP Cache Headers**: Control browser and CDN caching
- **Data Cache**: Next.js internal cache for fetch results

## Real World Context

Proper caching can:
- Reduce database queries by 90%+
- Cut response times from seconds to milliseconds
- Lower serverless costs dramatically
- Handle traffic spikes without scaling

## Deep Dive

### Route-Level Caching

```typescript
// Cache for 60 seconds
export const revalidate = 60;

export async function GET() {
  const data = await fetchData();
  return Response.json(data);
}
```

### Force Dynamic (No Cache)

```typescript
export const dynamic = 'force-dynamic';

export async function GET() {
  // Always fresh - good for user-specific data
  const session = await getSession();
  return Response.json({ user: session.user });
}
```

### Fetch-Level Caching

```typescript
// Cache external API calls
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 }, // Cache for 1 hour
});

// No cache for this specific fetch
const freshData = await fetch('https://api.example.com/user', {
  cache: 'no-store',
});
```

### HTTP Cache Headers

```typescript
export async function GET() {
  const data = await getPublicData();
  
  return new Response(JSON.stringify(data), {
    headers: {
      'Content-Type': 'application/json',
      // Browser caches for 60s, CDN caches for 1 hour
      // Serve stale while revalidating for up to 1 day
      'Cache-Control': 'public, max-age=60, s-maxage=3600, stale-while-revalidate=86400',
    },
  });
}
```

### Cache Header Explained

```
public              - Can be cached by browsers and CDNs
private             - Only browser can cache (user-specific data)
max-age=60          - Browser caches for 60 seconds
s-maxage=3600       - CDN caches for 1 hour
stale-while-revalidate=86400
                    - Serve stale while fetching fresh (1 day)
no-store            - Never cache
```

## Common Pitfalls

1. **Caching user-specific data**: Don't cache responses that contain user data with `public` headers.

2. **Forgetting `s-maxage`**: `max-age` alone doesn't tell CDNs to cache. Use both.

3. **Not testing cache behavior**: Use browser DevTools and `curl -I` to verify headers.

## Best Practices

- **Use `revalidate` for simple cases**: It handles most scenarios
- **Set appropriate durations**: Balance freshness vs performance
- **Use `stale-while-revalidate`**: Users see fast responses while cache refreshes
- **Never cache authenticated responses publicly**: Use `private` or `no-store`

## Summary

Next.js provides multiple caching options. Use `export const revalidate` for route-level caching, fetch options for external API caching, and HTTP headers for browser/CDN control. Always consider whether data is public or user-specific when choosing cache settings.

## Resources

- [Caching in Next.js](https://nextjs.org/docs/app/building-your-application/caching) — Comprehensive caching documentation

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*