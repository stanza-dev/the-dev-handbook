---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-edge-middleware"
---

# Edge Middleware Optimization

## Introduction

Middleware runs at the edge on every matched request. Optimizing it is critical—slow middleware means slow everything.

## Key Concepts

**Middleware performance tips**:

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
export function middleware(request: NextRequest) {
  // Good: Simple cookie check
  const token = request.cookies.get('token');
  if (!token) return NextResponse.redirect('/login');
  
  // Bad: Database lookup on every request
  // const user = await db.user.findUnique({ where: { token } });
  
  return NextResponse.next();
}
```

### Edge-Compatible Libraries

```typescript
// Use jose (Edge-compatible) not jsonwebtoken
import { jwtVerify } from 'jose';

const secret = new TextEncoder().encode(process.env.JWT_SECRET);

export async function middleware(request: NextRequest) {
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

Keep middleware fast by using efficient matchers, avoiding heavy computations, and using Edge-compatible libraries. Every millisecond in middleware adds to every request.

## Resources

- [Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware) — Middleware optimization tips

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*