---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-what-is-proxy"
---

# What is Proxy in Next.js 16?

## Introduction

Imagine having a security guard at your building's entrance who checks IDs, redirects visitors, and logs every entry. That's exactly what Proxy does for your Next.js application—it intercepts every request before it reaches your routes.

## Key Concepts

**Proxy** is a special file (`proxy.ts` or `proxy.js`) that runs code **before a request is completed**. It sits between the incoming request and your application's routes, giving you the power to:

- Inspect and modify requests
- Redirect users to different routes
- Rewrite URLs internally
- Add or modify headers and cookies

> **Note:** In Next.js 16, `middleware` has been renamed to `proxy` to better reflect its network boundary and routing focus.

## Real World Context

In production applications, Proxy is essential for:

- **Authentication Gates**: Redirect unauthenticated users to login before they can access `/dashboard`
- **Geo-based Routing**: Show different content based on user location
- **A/B Testing**: Route users to different page variants
- **Bot Protection**: Block or challenge suspicious traffic
- **Analytics**: Log request patterns without modifying your page code

## Deep Dive

### Creating Your First Proxy

Create a `proxy.ts` file in your project root (next to `app/`):

```typescript
// proxy.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function proxy(request: NextRequest) {
  console.log('Request to:', request.nextUrl.pathname);
  return NextResponse.next();
}
```

### Execution Flow

1. User makes a request to `/dashboard`
2. Proxy intercepts the request
3. Your proxy function runs and decides what to do
4. If `NextResponse.next()` is returned, the request continues to the route
5. If `NextResponse.redirect()` is returned, the user is sent elsewhere

### Runtime Environment

Proxy runs in the **Node.js runtime** by default, giving you access to:
- Full Node.js APIs
- Database connections
- File system (with limitations)

## Common Pitfalls

1. **Running on every request**: Without a `matcher` config, Proxy runs on ALL requests including static assets, dramatically slowing your site.

2. **Heavy computations**: Proxy adds latency to every matched request. Avoid database queries or heavy operations here—use caching or move logic to route handlers.

3. **Forgetting to return**: Always return `NextResponse.next()` to continue the request chain, or your app will hang.

## Best Practices

- **Always use matchers**: Limit Proxy to specific routes to avoid performance overhead
- **Keep it lightweight**: Proxy runs on every request—milliseconds matter
- **Use for routing decisions only**: Authentication checks, redirects, and header modifications—not business logic
- **Log sparingly**: Excessive logging in Proxy can fill your logs quickly

## Summary

Proxy is Next.js 16's request interceptor that runs before your routes. Place `proxy.ts` at your project root, use it for authentication, redirects, and request modifications, and always configure matchers to limit its scope. Remember: fast Proxy = fast app.

## Resources

- [Proxy Documentation](https://nextjs.org/docs/app/building-your-application/routing/middleware) — Official Next.js proxy guide
- [Next.js Routing Overview](https://nextjs.org/docs/app/building-your-application/routing) — Understanding how routing works in Next.js

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*