---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-request-inspection"
---

# Request Inspection in Proxy

## Introduction

Before you can make routing decisions, you need to know what you're working with. The `NextRequest` object is your window into every detail of the incoming request—headers, cookies, URL parts, and more.

## Key Concepts

**NextRequest** extends the standard Web `Request` API with Next.js-specific helpers:

- `request.nextUrl` - Parsed URL with pathname, search params, etc.
- `request.cookies` - Cookie jar with get/set/delete methods
- `request.headers` - Standard Headers object
- `request.geo` - Geolocation data (on Vercel)
- `request.ip` - Client IP address (on Vercel)

## Real World Context

Request inspection powers critical features:

- **Auth checks**: Read session cookies to verify authentication
- **Geo-routing**: Show country-specific content or redirect to regional sites
- **Bot detection**: Analyze User-Agent and request patterns
- **Feature flags**: Check cookies or headers for A/B test variants

## Deep Dive

### Accessing URL Information

```typescript
export function proxy(request: NextRequest) {
  const { pathname, searchParams } = request.nextUrl;
  
  console.log('Path:', pathname);           // /dashboard/settings
  console.log('Query:', searchParams.get('tab')); // 'profile'
  console.log('Full URL:', request.url);    // https://example.com/dashboard/settings?tab=profile
  
  return NextResponse.next();
}
```

### Reading Cookies

```typescript
export function proxy(request: NextRequest) {
  // Get a specific cookie
  const sessionToken = request.cookies.get('session')?.value;
  
  // Check if cookie exists
  const hasSession = request.cookies.has('session');
  
  // Get all cookies
  const allCookies = request.cookies.getAll();
  
  if (!sessionToken) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  return NextResponse.next();
}
```

### Inspecting Headers

```typescript
export function proxy(request: NextRequest) {
  const userAgent = request.headers.get('user-agent');
  const acceptLanguage = request.headers.get('accept-language');
  const authorization = request.headers.get('authorization');
  
  // Detect mobile devices
  const isMobile = /mobile/i.test(userAgent || '');
  
  if (isMobile) {
    return NextResponse.rewrite(new URL('/mobile' + request.nextUrl.pathname, request.url));
  }
  
  return NextResponse.next();
}
```

### Geolocation (Vercel)

```typescript
export function proxy(request: NextRequest) {
  const country = request.geo?.country || 'US';
  const city = request.geo?.city || 'Unknown';
  const region = request.geo?.region || 'Unknown';
  
  // Redirect EU users to GDPR-compliant pages
  const euCountries = ['DE', 'FR', 'IT', 'ES', 'NL'];
  if (euCountries.includes(country)) {
    return NextResponse.redirect(new URL('/eu' + request.nextUrl.pathname, request.url));
  }
  
  return NextResponse.next();
}
```

## Common Pitfalls

1. **Assuming cookies exist**: Always use optional chaining (`?.value`) when reading cookies—they may not be present.

2. **Case sensitivity**: Header names are case-insensitive, but cookie names are case-sensitive.

3. **Geo data availability**: `request.geo` and `request.ip` only work on Vercel. For self-hosted apps, parse the `x-forwarded-for` header.

## Best Practices

- **Validate cookie values**: Don't trust cookie contents—validate session tokens server-side
- **Use type guards**: Check for undefined/null before using request properties
- **Log minimally**: Avoid logging sensitive data like full cookies or authorization headers
- **Cache lookups**: If you need to check the same value multiple times, store it in a variable

## Summary

`NextRequest` gives you complete visibility into incoming requests. Use `nextUrl` for path information, `cookies` for session data, `headers` for client information, and `geo` for location-based routing. Always handle missing data gracefully with optional chaining and fallback values.

## Resources

- [NextRequest API Reference](https://nextjs.org/docs/app/api-reference/functions/next-request) — Full NextRequest API documentation

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*