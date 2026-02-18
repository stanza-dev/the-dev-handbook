---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-public-routes"
---

# Handling Public Routes

## Introduction

Not every route needs a security guard. Your marketing pages, login form, and documentation should be freely accessible. The art of route protection is knowing what to protect—and more importantly, what NOT to protect.

## Key Concepts

**Public routes** are paths accessible without authentication. These typically include:
- Authentication pages (`/login`, `/signup`, `/forgot-password`)
- Marketing pages (`/`, `/about`, `/pricing`)
- Legal pages (`/privacy`, `/terms`)
- API routes that handle authentication

**Auth-only public routes** are a special category—they should redirect authenticated users away (you don't need to see the login page if you're already logged in).

## Real World Context

Incorrect public route handling causes:
- Users locked out of the login page (infinite redirect loops)
- Authenticated users seeing login pages (confusing UX)
- SEO issues when crawlers can't access marketing pages
- Support tickets from frustrated users

## Deep Dive

### Strategy 1: Matcher Exclusion (Recommended)

The cleanest approach—only run proxy on protected paths:

```typescript
export const config = {
  matcher: ['/dashboard/:path*', '/account/:path*', '/settings/:path*'],
};
```

Routes not in the matcher (like `/`, `/login`) are automatically public and never trigger proxy.

### Strategy 2: Explicit Public Path List

For more control, check paths inside proxy:

```typescript
const publicPaths = [
  '/login',
  '/signup',
  '/forgot-password',
  '/reset-password',
  '/verify-email',
];

export function proxy(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Allow public paths without any checks
  if (publicPaths.some(path => pathname.startsWith(path))) {
    return NextResponse.next();
  }

  // Everything else requires authentication
  const token = request.cookies.get('session')?.value;
  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}
```

### Redirect Authenticated Users Away from Auth Pages

Don't show login pages to logged-in users:

```typescript
const authPaths = ['/login', '/signup'];

export function proxy(request: NextRequest) {
  const token = request.cookies.get('session')?.value;
  const { pathname } = request.nextUrl;

  // Redirect authenticated users away from auth pages
  if (token && authPaths.some(path => pathname.startsWith(path))) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }

  // Redirect unauthenticated users to login for protected routes
  if (!token && !authPaths.some(path => pathname.startsWith(path))) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}
```

### Combining Both Strategies

```typescript
const authOnlyPaths = ['/login', '/signup', '/forgot-password'];
const protectedPaths = ['/dashboard', '/settings', '/account'];

export function proxy(request: NextRequest) {
  const token = request.cookies.get('session')?.value;
  const { pathname } = request.nextUrl;

  // Logged-in users shouldn't see auth pages
  if (token && authOnlyPaths.some(p => pathname.startsWith(p))) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }

  // Non-logged-in users can't access protected pages
  if (!token && protectedPaths.some(p => pathname.startsWith(p))) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}
```

## Common Pitfalls

1. **Forgetting the root path**: If `/` needs protection but `/login` doesn't, your matcher logic needs to handle both cases.

2. **API route conflicts**: Don't protect `/api/auth/*` routes—they need to handle authentication themselves.

3. **Case sensitivity**: `/Login` and `/login` are different paths. Normalize paths if your app allows case variations.

## Best Practices

- **Prefer matcher exclusion**: It's clearer and faster than runtime path checking
- **Document your public paths**: Future developers need to know which routes are intentionally unprotected
- **Test auth redirects**: Verify logged-in users can't access login and vice versa
- **Consider API routes separately**: They often have different auth requirements

## Summary

Public routes don't need protection, but they do need intentional handling. Use matchers to exclude them from proxy, or explicitly check paths inside your proxy function. Always redirect authenticated users away from login pages to avoid confusion.

## Resources

- [Middleware Matching Paths](https://nextjs.org/docs/app/building-your-application/routing/middleware#matching-paths) — How to configure middleware path matching

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*