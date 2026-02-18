---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-dynamic-segments"
---

# Dynamic Segments

## Introduction

Static routes like `/api/users` only get you so far. To handle `/api/users/123` or `/api/posts/my-first-post`, you need dynamic segments—placeholders in your route that capture URL values.

## Key Concepts

**Dynamic segments** use bracket syntax:

- `[id]` - Single dynamic segment (one path part)
- `[...slug]` - Catch-all segment (one or more parts)
- `[[...slug]]` - Optional catch-all (zero or more parts)

## Real World Context

Dynamic routes power:
- Individual resource endpoints (`/api/users/:id`)
- Nested resources (`/api/posts/:postId/comments/:commentId`)
- File path handling (`/api/files/documents/report.pdf`)
- Flexible API versioning (`/api/v1/users`, `/api/v2/users`)

## Deep Dive

### Single Dynamic Segment

```typescript
// app/api/users/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const { id } = params;
  
  const user = await prisma.user.findUnique({
    where: { id },
  });
  
  if (!user) {
    return Response.json({ error: 'User not found' }, { status: 404 });
  }
  
  return Response.json(user);
}
// GET /api/users/abc123 → params.id = 'abc123'
```

### Multiple Dynamic Segments

```typescript
// app/api/posts/[postId]/comments/[commentId]/route.ts
export async function GET(
  request: Request,
  { params }: { params: { postId: string; commentId: string } }
) {
  const { postId, commentId } = params;
  
  const comment = await prisma.comment.findFirst({
    where: {
      id: commentId,
      postId: postId,
    },
  });
  
  return Response.json(comment);
}
// GET /api/posts/post-1/comments/comment-5
```

### Catch-All Segments

```typescript
// app/api/files/[...path]/route.ts
export async function GET(
  request: Request,
  { params }: { params: { path: string[] } }
) {
  const filePath = params.path.join('/');
  
  return Response.json({ path: filePath, segments: params.path });
}
// GET /api/files/documents/2024/report.pdf
// → params.path = ['documents', '2024', 'report.pdf']
```

### Optional Catch-All

```typescript
// app/api/docs/[[...slug]]/route.ts
export async function GET(
  request: Request,
  { params }: { params: { slug?: string[] } }
) {
  const slug = params.slug || [];
  
  if (slug.length === 0) {
    return Response.json({ page: 'index' });
  }
  
  return Response.json({ page: slug.join('/') });
}
// GET /api/docs → params.slug = undefined
// GET /api/docs/intro → params.slug = ['intro']
// GET /api/docs/api/reference → params.slug = ['api', 'reference']
```

### Type Safety

```typescript
type RouteParams = {
  params: {
    id: string;
  };
};

export async function GET(request: Request, { params }: RouteParams) {
  const { id } = params;
  // id is typed as string
}
```

## Common Pitfalls

1. **Params are always strings**: Even if the URL is `/api/users/123`, `params.id` is `'123'` (string), not `123` (number). Parse if needed.

2. **Case sensitivity**: `/api/Users/123` and `/api/users/123` are different routes. Be consistent.

3. **Forgetting the folder**: `[id]/route.ts` creates the dynamic segment, not just naming the file `[id].ts`.

## Best Practices

- **Use descriptive names**: `[userId]` is clearer than `[id]` when you have multiple dynamic segments
- **Validate params**: Check that IDs exist before querying the database
- **Parse numeric IDs**: `parseInt(params.id)` when working with numeric database IDs
- **Use catch-all sparingly**: Prefer explicit routes when possible

## Summary

Dynamic segments capture URL values in your route handlers. Use `[name]` for single segments, `[...name]` for catch-all, and `[[...name]]` for optional catch-all. Params are always strings and accessed via the second argument to your handler function.

## Resources

- [Dynamic Routes](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes) — Dynamic route segments documentation

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*