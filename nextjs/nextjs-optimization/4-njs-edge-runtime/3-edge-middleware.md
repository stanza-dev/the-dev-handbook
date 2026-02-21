---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-edge-middleware"
---

# Proxy Optimization

## Introduction

In Next.js 16, proxy (formerly middleware) runs on the Node.js runtime by default on every matched request. Optimizing it is critical—slow proxy logic means slow everything.

## Key Concepts

**Proxy performance tips**:

- Keep it lightweight (no heavy computations)
- Use efficient matching patterns
- Cache lookups when possible
- Minimize external calls

## Deep Dive

### Efficient Matching

```typescript
// Good: Specific matcher
export const config = {
  matcher: ['/dashboard/:path*', '/api/protected/:path*'],
};

// Bad: Runs on everything
export const config = {
  matcher: '/:path*',
};
```

### Lightweight Checks

```typescript
export function proxy(request: NextRequest) {
  // Good: Simple cookie check
  const token = request.cookies.get('token');
  if (!token) return NextResponse.redirect('/login');
  
  // Bad: Database lookup on every request
  // const user = await db.user.findUnique({ where: { token } });
  
  return NextResponse.next();
}
```

### Node.js Runtime by Default

In Next.js 16, proxy runs on the Node.js runtime by default, giving you access to the full Node.js API. You can opt into the Edge runtime if you need lower latency for simple operations:

```typescript
// proxy.ts
import { jwtVerify } from 'jose';

const secret = new TextEncoder().encode(process.env.JWT_SECRET);

export async function proxy(request: NextRequest) {
  const token = request.cookies.get('token')?.value;
  if (!token) return NextResponse.redirect('/login');
  
  try {
    await jwtVerify(token, secret);
    return NextResponse.next();
  } catch {
    return NextResponse.redirect('/login');
  }
}
```

## Summary

Keep proxy logic fast by using efficient matchers, avoiding heavy computations, and leveraging the Node.js runtime for full API access. Every millisecond in proxy adds to every request.

## Resources

- [Proxy](https://nextjs.org/docs/app/building-your-application/routing/proxy) — Proxy optimization tips

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*