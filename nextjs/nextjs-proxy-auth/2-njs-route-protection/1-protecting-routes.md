---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-protecting-routes"
---

# Protecting Routes with Proxy

## Introduction

Every production app has routes that only certain users should access. Whether it's a dashboard, admin panel, or user settings, route protection is your first defense against unauthorized access. Proxy makes this elegant and performant.

## Key Concepts

**Route protection** is the practice of restricting access to certain URL paths based on authentication or authorization status. In Next.js, this happens at the proxy layer—before the page even starts rendering.

The protection flow:
1. User requests a protected route
2. Proxy checks for valid authentication (usually a cookie)
3. If valid → allow access
4. If invalid → redirect to login

## Real World Context

Without route protection:
- Users could access `/dashboard` by typing the URL directly
- Sensitive data could be exposed during the initial page load
- Server resources would be wasted rendering pages for unauthorized users

Proxy-level protection means unauthenticated users never even see a flash of protected content.

## Deep Dive

### Basic Protection Pattern

```typescript
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function proxy(request: NextRequest) {
  const token = request.cookies.get('session')?.value;
  const { pathname } = request.nextUrl;

  // Define protected routes
  const protectedPaths = ['/dashboard', '/settings', '/account'];
  const isProtected = protectedPaths.some(path => pathname.startsWith(path));

  // If no token and trying to access protected route
  if (!token && isProtected) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('from', pathname);
    return NextResponse.redirect(loginUrl);
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/settings/:path*', '/account/:path*'],
};
```

### Preserving the Return URL

The `from` query parameter remembers where the user wanted to go:

```typescript
// In proxy - save original destination
loginUrl.searchParams.set('from', pathname);

// In login page - redirect back after success
const from = searchParams.get('from') || '/dashboard';
router.push(from);
```

### Token Validation

Simply checking for a cookie's existence isn't enough for production:

```typescript
import { jwtVerify } from 'jose';

const secret = new TextEncoder().encode(process.env.JWT_SECRET);

export async function proxy(request: NextRequest) {
  const token = request.cookies.get('session')?.value;

  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  try {
    // Actually verify the token is valid
    await jwtVerify(token, secret);
    return NextResponse.next();
  } catch {
    // Token is invalid or expired
    const response = NextResponse.redirect(new URL('/login', request.url));
    response.cookies.delete('session'); // Clear invalid cookie
    return response;
  }
}
```

## Common Pitfalls

1. **Only checking cookie existence**: A malicious user could set any cookie value. Always validate tokens server-side.

2. **Redirect loops**: If `/login` is accidentally included in protected routes, users get stuck in an infinite redirect loop.

3. **Forgetting the matcher**: Without a proper matcher, proxy runs on every request, including the login page itself.

## Best Practices

- **Always validate tokens**: Don't just check if a cookie exists—verify its signature and expiration
- **Use HTTPS in production**: Cookies should have the `secure` flag set
- **Clear invalid cookies**: If a token fails validation, delete it to prevent repeated failures
- **Preserve query parameters**: Include the full URL (with query params) in the `from` parameter

## Summary

Route protection with proxy ensures unauthenticated users are redirected before any protected content loads. Check for session tokens, validate them properly, and always preserve the original URL for post-login redirects. Use matchers to limit proxy execution to protected routes only.

## Resources

- [Authentication Guide](https://nextjs.org/docs/app/building-your-application/authentication) — Official Next.js authentication patterns

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*