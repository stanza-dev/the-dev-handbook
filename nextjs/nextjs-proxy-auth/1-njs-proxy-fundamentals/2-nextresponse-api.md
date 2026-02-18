---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-nextresponse-api"
---

# The NextResponse API

## Introduction

Every proxy function needs to tell Next.js what to do with the request: let it through, redirect it, or modify it. `NextResponse` is your toolkit for making these decisions with a clean, chainable API.

## Key Concepts

**NextResponse** extends the standard Web `Response` API with convenience methods specifically designed for proxy operations:

- `NextResponse.next()` - Continue to the route
- `NextResponse.redirect()` - Send user to a different URL
- `NextResponse.rewrite()` - Internally route to a different path
- `NextResponse.json()` - Return JSON directly from proxy

## Real World Context

Think of `NextResponse` as a traffic controller at an intersection:

- **Green light** (`next()`) - Let the request proceed normally
- **Detour sign** (`redirect()`) - Send the user to a completely different destination
- **Hidden tunnel** (`rewrite()`) - Route internally without the user knowing

Without mastering `NextResponse`, you cannot build authentication, authorization, or any request modification logic.

## Deep Dive

### Continuing the Request

```typescript
// Let the request proceed to its original destination
return NextResponse.next();
```

### Redirecting Users

```typescript
// Temporary redirect (307) - default
return NextResponse.redirect(new URL('/login', request.url));

// Permanent redirect (308)
return NextResponse.redirect(new URL('/new-page', request.url), {
  status: 308,
});
```

### URL Rewriting

Rewrites are invisible to the user—the browser URL stays the same:

```typescript
// User sees /old-blog/post-1, but server renders /blog/post-1
if (request.nextUrl.pathname.startsWith('/old-blog')) {
  return NextResponse.rewrite(
    new URL(request.nextUrl.pathname.replace('/old-blog', '/blog'), request.url)
  );
}
```

### Modifying Headers

```typescript
const response = NextResponse.next();

// Add custom headers
response.headers.set('x-request-id', crypto.randomUUID());
response.headers.set('x-custom-header', 'my-value');

// Pass user info to server components
response.headers.set('x-user-id', userId);

return response;
```

### Managing Cookies

```typescript
const response = NextResponse.next();

// Set a cookie
response.cookies.set('session', 'abc123', {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'lax',
  maxAge: 60 * 60 * 24 * 7, // 1 week
});

// Delete a cookie
response.cookies.delete('old-session');

return response;
```

## Common Pitfalls

1. **Forgetting to use `new URL()`**: Redirect and rewrite require URL objects, not strings. Always construct URLs properly.

2. **Redirect loops**: Redirecting to a path that triggers the same proxy logic creates an infinite loop. Always exclude your redirect destinations from the matcher.

3. **Losing request context**: When creating a new URL, use `request.url` as the base to preserve the host and protocol.

## Best Practices

- **Chain modifications**: Create the response once, modify it, then return it
- **Use absolute URLs for external redirects**: `NextResponse.redirect('https://external.com')`
- **Preserve query parameters**: Include `request.nextUrl.search` when rewriting
- **Set secure cookie flags**: Always use `httpOnly` and `secure` for sensitive cookies

## Summary

`NextResponse` is your primary tool for controlling request flow in proxy. Use `next()` to proceed, `redirect()` for visible URL changes, and `rewrite()` for invisible routing. Master header and cookie manipulation to pass data between proxy and your routes.

## Resources

- [NextResponse API Reference](https://nextjs.org/docs/app/api-reference/functions/next-response) — Full API reference for NextResponse
- [Cookies in Next.js](https://nextjs.org/docs/app/api-reference/functions/cookies) — Working with cookies in Next.js

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*