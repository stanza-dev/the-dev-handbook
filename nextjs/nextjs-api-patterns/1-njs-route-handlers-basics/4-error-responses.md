---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-error-responses"
---

# Error Responses

## Introduction

APIs fail—invalid input, missing resources, server errors. How you handle and communicate these failures determines whether your API is a joy or frustration to work with.

## Key Concepts

**HTTP Status Code Categories**:

- **2xx Success**: Request succeeded
- **4xx Client Error**: Client made a mistake
- **5xx Server Error**: Server failed to fulfill a valid request

**Common Status Codes**:

| Code | Name | Use Case |
|------|------|----------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Missing authentication |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate entry |
| 422 | Unprocessable Entity | Validation failed |
| 500 | Internal Server Error | Server crashed |

## Real World Context

Good error responses enable:
- Frontend apps to show appropriate error messages
- API consumers to handle failures gracefully
- Debugging by providing helpful context
- Security by not leaking internal details

## Deep Dive

### Consistent Error Format

```typescript
type ErrorResponse = {
  error: string;           // Human-readable message
  code?: string;           // Machine-readable error code
  details?: Record<string, string[]>; // Field-specific errors
};

function errorResponse(
  message: string,
  status: number,
  details?: Record<string, string[]>
) {
  return Response.json(
    { error: message, details },
    { status }
  );
}
```

### Input Validation Errors (400/422)

```typescript
export async function POST(request: Request) {
  const body = await request.json();
  
  const errors: Record<string, string[]> = {};
  
  if (!body.email) {
    errors.email = ['Email is required'];
  } else if (!isValidEmail(body.email)) {
    errors.email = ['Invalid email format'];
  }
  
  if (!body.password) {
    errors.password = ['Password is required'];
  } else if (body.password.length < 8) {
    errors.password = ['Password must be at least 8 characters'];
  }
  
  if (Object.keys(errors).length > 0) {
    return Response.json(
      { error: 'Validation failed', details: errors },
      { status: 422 }
    );
  }
  
  // Continue with valid data...
}
```

### Not Found (404)

```typescript
export async function GET(
  request: Request,
  props: { params: Promise<{ id: string }> }
) {
  const { id } = await props.params;
  const user = await prisma.user.findUnique({
    where: { id },
  });
  
  if (!user) {
    return Response.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }
  
  return Response.json(user);
}
```

### Conflict (409)

```typescript
export async function POST(request: Request) {
  const { email } = await request.json();
  
  const existing = await prisma.user.findUnique({
    where: { email },
  });
  
  if (existing) {
    return Response.json(
      { error: 'A user with this email already exists' },
      { status: 409 }
    );
  }
  
  // Create user...
}
```

### Catching Unexpected Errors (500)

```typescript
export async function GET() {
  try {
    const data = await fetchExternalAPI();
    return Response.json(data);
  } catch (error) {
    console.error('API Error:', error);
    
    // Don't expose internal error details
    return Response.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Creating an Error Handler Utility

```typescript
// lib/api-errors.ts
export class APIError extends Error {
  constructor(
    message: string,
    public status: number,
    public code?: string
  ) {
    super(message);
  }
}

export function handleAPIError(error: unknown) {
  if (error instanceof APIError) {
    return Response.json(
      { error: error.message, code: error.code },
      { status: error.status }
    );
  }
  
  console.error('Unexpected error:', error);
  return Response.json(
    { error: 'Internal server error' },
    { status: 500 }
  );
}

// Usage
export async function GET() {
  try {
    const user = await getUser(id);
    if (!user) throw new APIError('User not found', 404, 'USER_NOT_FOUND');
    return Response.json(user);
  } catch (error) {
    return handleAPIError(error);
  }
}
```

## Common Pitfalls

1. **Leaking stack traces**: Never send `error.stack` or database errors to clients. Log them server-side.

2. **Using 200 for errors**: Return proper status codes. Clients rely on them for error handling.

3. **Inconsistent error format**: Always return `{ error: string }` so clients know what to expect.

## Best Practices

- **Use specific status codes**: 422 for validation, 409 for conflicts, not just 400
- **Include error codes**: Machine-readable codes like `USER_NOT_FOUND` help with i18n
- **Provide actionable messages**: "Email is required" beats "Invalid input"
- **Log server errors**: Track 500s for debugging but don't expose details to clients

## Summary

Good error responses use appropriate HTTP status codes, consistent JSON format, and helpful messages without leaking sensitive information. Create utility functions to standardize error handling across your API, and always log unexpected errors for debugging.

## Resources

- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) — MDN HTTP Status Codes reference

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*