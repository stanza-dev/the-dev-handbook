---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-crud-operations"
---

# Complete CRUD Example

A full RESTful API for a resource.

## GET All and POST

```typescript
// app/api/posts/route.ts
import { prisma } from '@/lib/prisma';

export async function GET() {
  const posts = await prisma.post.findMany();
  return Response.json(posts);
}

export async function POST(request: Request) {
  const data = await request.json();
  const post = await prisma.post.create({ data });
  return Response.json(post, { status: 201 });
}
```

## GET One, PUT, DELETE

```typescript
// app/api/posts/[id]/route.ts
import { prisma } from '@/lib/prisma';

export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const post = await prisma.post.findUnique({
    where: { id: parseInt(params.id) },
  });

  if (!post) {
    return Response.json({ error: 'Not found' }, { status: 404 });
  }

  return Response.json(post);
}

export async function PUT(
  request: Request,
  { params }: { params: { id: string } }
) {
  const data = await request.json();
  const post = await prisma.post.update({
    where: { id: parseInt(params.id) },
    data,
  });
  return Response.json(post);
}

export async function DELETE(
  request: Request,
  { params }: { params: { id: string } }
) {
  await prisma.post.delete({
    where: { id: parseInt(params.id) },
  });
  return new Response(null, { status: 204 });
}
```

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*