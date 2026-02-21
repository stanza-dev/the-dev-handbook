---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-edge-use-cases"
---

# When to Use Edge

## Good Use Cases

1. **Personalization**: Geo-based content, A/B testing.
2. **Redirects**: Fast redirects based on cookies or headers.
3. **Simple APIs**: Lightweight endpoints that don't need node APIs.
4. **Proxy**: By default, proxy runs on the Node.js runtime in Next.js 16, with an option to use the Edge runtime.

## Example: Geo-Based Content

```typescript
export const runtime = 'edge';

export async function GET(request: Request) {
  const country = request.headers.get('x-vercel-ip-country') || 'US';
  
  const content = {
    US: 'Hello, America!',
    UK: 'Hello, United Kingdom!',
    DE: 'Hallo, Deutschland!',
  };

  return Response.json({
    message: content[country] || 'Hello, World!',
    country,
  });
}
```

## When NOT to Use Edge

- Database connections (use serverless functions instead).
- Heavy computation.
- File system access.
- Native Node modules.

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*