---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-edge-intro"
---

# Edge Runtime

The Edge Runtime is a lightweight JavaScript runtime that runs code closer to your users, at the edge of the network.

## Benefits

- **Lower latency**: Code runs in data centers near users.
- **Faster cold starts**: Lighter than Node.js.
- **Global distribution**: Automatic worldwide deployment.

## Enabling Edge Runtime

For Route Handlers:

```typescript
export const runtime = 'edge';

export async function GET() {
  return Response.json({ message: 'Hello from the edge!' });
}
```

For Pages/Layouts:

```typescript
export const runtime = 'edge';

export default function Page() {
  return <h1>Edge-rendered page</h1>;
}
```

## Limitations

The Edge Runtime doesn't have access to all Node.js APIs:

- ❌ `fs` (filesystem)
- ❌ `child_process`
- ❌ Native Node modules
- ✅ `fetch`
- ✅ `crypto.subtle`
- ✅ `TextEncoder/Decoder`

## Resources

- [Edge Runtime](https://nextjs.org/docs/app/building-your-application/rendering/edge-and-nodejs-runtimes) — Edge vs Node.js runtime comparison

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*