---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-edge-vs-node"
---

# Choosing Edge vs Node Runtime

## Introduction

Not every route should run at the Edge. Understanding when to use each runtime helps you make the right performance and capability tradeoffs.

## Key Concepts

**Choose Edge when**:
- Low latency is critical
- Logic is simple (auth checks, redirects)
- No Node.js-specific APIs needed

**Choose Node.js when**:
- Database connections required
- File system access needed
- Heavy computation
- Node.js libraries required

## Deep Dive

### Edge Route Handler

```typescript
// app/api/geo/route.ts
export const runtime = 'edge';

export async function GET(request: Request) {
  // Simple, fast, no DB needed
  const country = request.headers.get('x-vercel-ip-country');
  return Response.json({ country });
}
```

### Node.js Route Handler

```typescript
// app/api/users/route.ts
// Default runtime is Node.js

export async function GET() {
  // Needs database connection
  const users = await prisma.user.findMany();
  return Response.json(users);
}
```

### Mixed Approach

```typescript
// Proxy for auth check (runs on Node.js by default in Next.js 16)
// Edge runtime for latency-sensitive route handlers
// Node.js for data fetching (route handlers)
```

## Summary

Proxy runs on the Node.js runtime by default in Next.js 16, giving you full Node.js API access. Use Edge runtime for latency-sensitive, simple route handlers. Use Node.js when you need database connections, file access, or Node-specific libraries.

## Resources

- [Runtimes](https://nextjs.org/docs/app/building-your-application/rendering/edge-and-nodejs-runtimes) — Edge vs Node.js runtime comparison

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*