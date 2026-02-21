---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-proxy-matchers"
---

# Proxy Matchers

## Introduction

Without matchers, your proxy runs on every single request—including images, CSS files, and fonts. That's like having a security guard check IDs for every piece of furniture delivery. Matchers let you target exactly which routes need inspection.

## Key Concepts

**Matchers** are path patterns that determine when your proxy function executes. They use a syntax similar to Express.js route patterns:

- `/dashboard` - Exact match
- `/dashboard/:path` - Matches one segment (e.g., `/dashboard/settings`)
- `/dashboard/:path*` - Matches zero or more segments (e.g., `/dashboard`, `/dashboard/a/b/c`)
- `/((?!api|_next).*)` - Regex with negative lookahead

## Real World Context

In production, running proxy on static assets:

- Increases latency for every image, font, and CSS file
- Wastes server resources on unnecessary checks
- Can break certain CDN optimizations

Proper matchers ensure proxy only runs where needed—typically on dynamic routes requiring authentication.

## Deep Dive

### Basic Matcher

```typescript
export const config = {
  matcher: '/dashboard/:path*',
};
```

This runs proxy only on `/dashboard` and all nested routes.

### Multiple Matchers

```typescript
export const config = {
  matcher: [
    '/dashboard/:path*',
    '/admin/:path*',
    '/api/protected/:path*',
    '/account/:path*',
  ],
};
```

### Excluding Static Assets (Recommended Pattern)

```typescript
export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico|.*\\.png$).*)',
  ],
};
```

This negative lookahead regex runs proxy on all paths **except**:
- API routes (`/api/*`)
- Static files (`/_next/static/*`)
- Optimized images (`/_next/image/*`)
- Favicon and PNG files

### Conditional Matching

For complex logic, use header or cookie-based matching:

```typescript
export const config = {
  matcher: [
    {
      source: '/dashboard/:path*',
      has: [
        { type: 'cookie', key: 'session' },
      ],
    },
  ],
};
```

## Common Pitfalls

1. **Forgetting to exclude `_next`**: Running proxy on Next.js internal routes (`/_next/*`) breaks static asset serving and dramatically slows your app.

2. **Overly broad regex**: A pattern like `/:path*` matches EVERYTHING. Be specific about what you want to match.

3. **Not testing matchers**: Use `console.log(request.nextUrl.pathname)` during development to verify which routes trigger your proxy.

## Best Practices

- **Start restrictive**: Begin with specific routes, then expand if needed
- **Always exclude static assets**: Use the negative lookahead pattern as a baseline
- **Document your matchers**: Complex regex patterns should have comments explaining their purpose
- **Test in development**: Verify your matchers work before deploying

## Summary

Matchers are your first line of defense against proxy performance issues. Use them to limit proxy execution to routes that actually need it. Always exclude `_next`, static assets, and public files. A well-configured matcher is the difference between a fast and slow Next.js app.

## Resources

- [Middleware Matcher](https://nextjs.org/docs/app/api-reference/file-conventions/proxy#matcher) — Official matcher configuration guide

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*