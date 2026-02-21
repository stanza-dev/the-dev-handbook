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
  props: { params: Promise<{ id: string }> }
) {
  const { id } = await props.params;
  
  const user = await prisma.user.findUnique({
    where: { id },
  });
  
  if (!user) {
    return Response.json({ error: 'User not found' }, { status: 404 });
  }
  
  return Response.json(user);
}
// GET /api/users/abc123 → id = 'abc123'
```

### Multiple Dynamic Segments

```typescript
// app/api/posts/[postId]/comments/[commentId]/route.ts
export async function GET(
  request: Request,
  props: { params: Promise<{ postId: string; commentId: string }> }
) {
  const { postId, commentId } = await props.params;
  
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
  props: { params: Promise<{ path: string[] }> }
) {
  const { path } = await props.params;
  const filePath = path.join('/');
  
  return Response.json({ path: filePath, segments: path });
}
// GET /api/files/documents/2024/report.pdf
// → path = ['documents', '2024', 'report.pdf']
```

### Optional Catch-All

```typescript
// app/api/docs/[[...slug]]/route.ts
export async function GET(
  request: Request,
  props: { params: Promise<{ slug?: string[] }> }
) {
  const { slug } = await props.params;
  const segments = slug || [];
  
  if (segments.length === 0) {
    return Response.json({ page: 'index' });
  }
  
  return Response.json({ page: segments.join('/') });
}
// GET /api/docs → slug = undefined
// GET /api/docs/intro → slug = ['intro']
// GET /api/docs/api/reference → slug = ['api', 'reference']
```

### Type Safety

```typescript
type RouteProps = {
  params: Promise<{
    id: string;
  }>;
};

export async function GET(request: Request, props: RouteProps) {
  const { id } = await props.params;
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

Dynamic segments capture URL values in your route handlers. Use `[name]` for single segments, `[...name]` for catch-all, and `[[...name]]` for optional catch-all. Params are always strings and accessed by awaiting `props.params` from the second argument to your handler function.

## Resources

- [Dynamic Routes](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes) — Dynamic route segments documentation

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*