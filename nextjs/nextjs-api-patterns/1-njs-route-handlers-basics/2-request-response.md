---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-request-response"
---

# Request and Response APIs

## Introduction

Route Handlers use the Web Platform's `Request` and `Response` APIs—the same APIs used by Service Workers and Cloudflare Workers. Mastering these APIs is essential for building robust APIs.

## Key Concepts

**Request** represents the incoming HTTP request:
- `request.url` - Full URL including query parameters
- `request.method` - HTTP method (GET, POST, etc.)
- `request.headers` - Request headers
- `request.json()` - Parse JSON body
- `request.text()` - Get body as text
- `request.formData()` - Parse form data

**Response** represents what you send back:
- `Response.json()` - Send JSON
- `new Response(body, options)` - Custom response

## Real World Context

You'll use these APIs for:
- Reading query parameters for filtering/pagination
- Parsing JSON bodies from form submissions
- Reading auth headers for API authentication
- Setting cache headers for performance
- Returning appropriate status codes

## Deep Dive

### Reading Query Parameters

```typescript
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '10');
  const search = searchParams.get('q');
  
  return Response.json({ page, limit, search });
}
// GET /api/users?page=2&limit=20&q=john
```

### Parsing JSON Body

```typescript
export async function POST(request: Request) {
  const body = await request.json();
  
  // body is typed as `any` - validate it!
  const { name, email } = body;
  
  if (!name || !email) {
    return Response.json(
      { error: 'Name and email are required' },
      { status: 400 }
    );
  }
  
  return Response.json({ received: { name, email } }, { status: 201 });
}
```

### Reading Headers

```typescript
import { headers } from 'next/headers';

export async function GET() {
  const headersList = await headers();
  
  const userAgent = headersList.get('user-agent');
  const authorization = headersList.get('authorization');
  const contentType = headersList.get('content-type');
  
  return Response.json({ userAgent });
}
```

### Building Responses

```typescript
// Simple JSON response
return Response.json({ data: 'value' });

// JSON with status code
return Response.json(
  { error: 'Not found' },
  { status: 404 }
);

// Custom headers
return new Response(JSON.stringify({ data: 'value' }), {
  status: 200,
  headers: {
    'Content-Type': 'application/json',
    'Cache-Control': 'public, max-age=3600',
    'Access-Control-Allow-Origin': '*',
  },
});

// No content response
return new Response(null, { status: 204 });

// Redirect
return Response.redirect('https://example.com', 302);
```

### Next.js Extended Headers

```typescript
import { headers, cookies } from 'next/headers';

export async function GET() {
  // Read cookies
  const cookieStore = await cookies();
  const token = cookieStore.get('token')?.value;
  
  // Read headers
  const headersList = await headers();
  const ip = headersList.get('x-forwarded-for');
  
  return Response.json({ hasToken: !!token, ip });
}
```

## Common Pitfalls

1. **Calling `.json()` twice**: The body stream can only be read once. Store the result if you need it multiple times.

2. **Missing `await`**: Both `request.json()` and `request.formData()` are async. Don't forget `await`.

3. **Case-sensitive headers**: Header names should be lowercase when using `.get()` for consistency.

## Best Practices

- **Validate input early**: Check required fields before processing
- **Use appropriate status codes**: 200 OK, 201 Created, 400 Bad Request, 404 Not Found, 500 Server Error
- **Set cache headers**: Help CDNs and browsers cache your responses
- **Return consistent error format**: Always return `{ error: string }` for errors

## Summary

Route Handlers use standard Web APIs for requests and responses. Read query params from `new URL(request.url).searchParams`, parse bodies with `await request.json()`, and access headers with Next.js's `await headers()` function. Build responses with `Response.json()` for convenience or `new Response()` for full control.

## Resources

- [Request API](https://developer.mozilla.org/en-US/docs/Web/API/Request) — MDN Request API reference
- [Response API](https://developer.mozilla.org/en-US/docs/Web/API/Response) — MDN Response API reference

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*