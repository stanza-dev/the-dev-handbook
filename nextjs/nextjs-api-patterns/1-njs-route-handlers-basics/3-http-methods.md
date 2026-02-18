---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-http-methods"
---

# HTTP Methods and REST Patterns

## Introduction

REST (Representational State Transfer) is the dominant pattern for API design. Understanding HTTP methods and how they map to CRUD operations is fundamental to building intuitive APIs.

## Key Concepts

**HTTP Methods** map to operations:

| Method | CRUD | Description | Idempotent |
|--------|------|-------------|------------|
| GET | Read | Retrieve resources | Yes |
| POST | Create | Create new resource | No |
| PUT | Update | Replace entire resource | Yes |
| PATCH | Update | Partial update | No |
| DELETE | Delete | Remove resource | Yes |

**Idempotent** means calling it multiple times has the same effect as calling it once.

## Real World Context

Proper HTTP method usage enables:
- Browser caching for GET requests
- Retry safety for idempotent operations
- Clear API contracts for consumers
- Correct behavior with proxies and CDNs

## Deep Dive

### GET - Retrieve Resources

```typescript
// app/api/users/route.ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get('page') || '1');
  
  const users = await prisma.user.findMany({
    skip: (page - 1) * 10,
    take: 10,
  });
  
  return Response.json(users);
}
```

### POST - Create Resources

```typescript
export async function POST(request: Request) {
  const data = await request.json();
  
  const user = await prisma.user.create({ data });
  
  // Return 201 Created with the new resource
  return Response.json(user, { status: 201 });
}
```

### PUT - Replace Resources

```typescript
// app/api/users/[id]/route.ts
export async function PUT(
  request: Request,
  { params }: { params: { id: string } }
) {
  const data = await request.json();
  
  // PUT replaces the entire resource
  const user = await prisma.user.update({
    where: { id: params.id },
    data: {
      name: data.name,
      email: data.email,
      role: data.role,
      // All fields must be provided
    },
  });
  
  return Response.json(user);
}
```

### PATCH - Partial Update

```typescript
export async function PATCH(
  request: Request,
  { params }: { params: { id: string } }
) {
  const data = await request.json();
  
  // PATCH only updates provided fields
  const user = await prisma.user.update({
    where: { id: params.id },
    data, // Only fields in `data` are updated
  });
  
  return Response.json(user);
}
```

### DELETE - Remove Resources

```typescript
export async function DELETE(
  request: Request,
  { params }: { params: { id: string } }
) {
  await prisma.user.delete({
    where: { id: params.id },
  });
  
  // 204 No Content - successful deletion with no body
  return new Response(null, { status: 204 });
}
```

### Complete CRUD File Structure

```typescript
// app/api/posts/route.ts - Collection endpoints
export async function GET() { /* List all posts */ }
export async function POST() { /* Create new post */ }

// app/api/posts/[id]/route.ts - Item endpoints
export async function GET() { /* Get single post */ }
export async function PUT() { /* Replace post */ }
export async function PATCH() { /* Update post */ }
export async function DELETE() { /* Delete post */ }
```

## Common Pitfalls

1. **Using POST for updates**: POST should create new resources. Use PUT or PATCH for updates.

2. **GET with body**: GET requests should not have a body. Use query parameters instead.

3. **Wrong status codes**: Return 201 for creation, 204 for deletion, not 200 for everything.

## Best Practices

- **Use nouns for resources**: `/api/users` not `/api/getUsers`
- **Use plural names**: `/api/posts` not `/api/post`
- **PUT requires all fields**: If a field is missing, it should be set to null
- **PATCH for partial updates**: Only update what's provided
- **DELETE is idempotent**: Deleting a non-existent resource should return 204, not 404

## Summary

HTTP methods map to CRUD operations: GET for reading, POST for creating, PUT for replacing, PATCH for updating, DELETE for removing. Use proper status codes (201 Created, 204 No Content) and follow REST conventions for intuitive, cacheable, and predictable APIs.

## Resources

- [HTTP Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) — MDN HTTP Methods reference

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*