---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-api-route-protection"
---

# Protecting API Routes

## Introduction

Your API routes are just as vulnerable as your pages—maybe more so. A determined user can call your API directly with tools like curl or Postman. Proxy-level protection ensures your API endpoints are guarded at the network boundary.

## Key Concepts

**API route protection** differs from page protection:
- Pages redirect to login when unauthorized
- APIs return HTTP error codes (401, 403)
- APIs may use different authentication methods (API keys, Bearer tokens)

## Real World Context

Unprotected API routes allow:
- Data theft (anyone can fetch user data)
- Unauthorized actions (deleting resources)
- Rate limit bypass (automated attacks)
- Enumeration attacks (probing for valid IDs)

## Deep Dive

### Protecting API Routes in Proxy

```typescript
export function proxy(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Handle API routes differently
  if (pathname.startsWith('/api/protected')) {
    const token = request.cookies.get('session')?.value;
    
    if (!token) {
      // Return 401 instead of redirecting
      return NextResponse.json(
        { error: 'Authentication required' },
        { status: 401 }
      );
    }
  }

  // Page routes get redirects
  if (pathname.startsWith('/dashboard')) {
    const token = request.cookies.get('session')?.value;
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }

  return NextResponse.next();
}
```

### Bearer Token Authentication

For APIs consumed by external services:

```typescript
export function proxy(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/api/v1')) {
    const authHeader = request.headers.get('authorization');
    
    if (!authHeader?.startsWith('Bearer ')) {
      return NextResponse.json(
        { error: 'Missing or invalid Authorization header' },
        { status: 401 }
      );
    }

    const token = authHeader.split(' ')[1];
    
    // Validate token (simplified example)
    if (token !== process.env.API_SECRET_KEY) {
      return NextResponse.json(
        { error: 'Invalid API key' },
        { status: 403 }
      );
    }
  }

  return NextResponse.next();
}
```

### Passing User Info to API Routes

After validation, pass user info via headers:

```typescript
export async function proxy(request: NextRequest) {
  const token = request.cookies.get('session')?.value;
  
  if (token) {
    try {
      const { payload } = await jwtVerify(token, secret);
      
      // Pass user info to the route handler
      const response = NextResponse.next();
      response.headers.set('x-user-id', payload.userId as string);
      response.headers.set('x-user-role', payload.role as string);
      return response;
    } catch {
      // Invalid token
    }
  }

  return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
}
```

Then in your Route Handler:

```typescript
export async function GET(request: Request) {
  const userId = request.headers.get('x-user-id');
  const userRole = request.headers.get('x-user-role');
  
  // Use the verified user info
  const data = await fetchUserData(userId);
  return Response.json(data);
}
```

## Common Pitfalls

1. **Redirecting API requests**: APIs should return JSON errors, not HTML redirects. Check the path before deciding the response type.

2. **Exposing internal errors**: Don't leak stack traces or database errors in API responses.

3. **Missing CORS headers**: If your API serves other domains, you may need to handle OPTIONS requests.

## Best Practices

- **Return proper status codes**: 401 for missing auth, 403 for insufficient permissions
- **Use consistent error format**: Always return JSON with an `error` field
- **Rate limit sensitive endpoints**: Combine with rate limiting for login and data endpoints
- **Log authentication failures**: Track failed auth attempts for security monitoring

## Summary

API routes need protection too, but with different responses—JSON errors instead of redirects. Use the Authorization header for external APIs and cookies for browser-based calls. Pass validated user info via headers so route handlers don't need to re-verify tokens.

## Resources

- [Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers) — Building APIs with Route Handlers

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*