---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-on-demand-revalidation"
---

# On-Demand Revalidation

Instead of waiting for a cache to expire, trigger revalidation programmatically.

## Using revalidatePath

Purge the cache for a specific path:

```typescript
import { revalidatePath } from 'next/cache';

export async function createPost(data: FormData) {
  'use server';
  
  await db.post.create({ data });
  
  // Revalidate the posts page
  revalidatePath('/posts');
}
```

## Using revalidateTag

Tag your fetches and revalidate by tag:

```typescript
// In your fetch
const posts = await fetch('https://api.example.com/posts', {
  next: { tags: ['posts'] },
});

// To revalidate
import { revalidateTag } from 'next/cache';
revalidateTag('posts');
```

## Webhook Revalidation

Create a Route Handler that your CMS can call:

```typescript
// app/api/revalidate/route.ts
import { revalidatePath } from 'next/cache';

export async function POST(request: Request) {
  const { path, secret } = await request.json();

  if (secret !== process.env.REVALIDATION_SECRET) {
    return Response.json({ error: 'Invalid secret' }, { status: 401 });
  }

  revalidatePath(path);
  return Response.json({ revalidated: true });
}
```

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*