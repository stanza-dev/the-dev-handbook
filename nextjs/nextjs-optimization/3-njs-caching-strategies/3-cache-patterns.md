---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-cache-patterns"
---

# Advanced Caching Patterns

## Introduction

Beyond basic ISR, Next.js 16 offers sophisticated caching with cache tags, unstable_cache, and the cacheLife directive for fine-grained control.

## Key Concepts

**Cache tags** enable:
- Group related data under a single tag
- Invalidate multiple caches at once
- Precise cache control for complex apps

## Deep Dive

### Using unstable_cache

```typescript
import { unstable_cache } from 'next/cache';

const getCachedUser = unstable_cache(
  async (userId: string) => {
    return await prisma.user.findUnique({ where: { id: userId } });
  },
  ['user'],
  {
    tags: ['users'],
    revalidate: 3600,
  }
);

// Usage
const user = await getCachedUser(userId);
```

### Tag-Based Invalidation

```typescript
// Fetch with tags
const posts = await fetch('https://api.example.com/posts', {
  next: { tags: ['posts', `user-${userId}`] },
});

// Invalidate all posts
import { revalidateTag } from 'next/cache';
revalidateTag('posts');

// Invalidate specific user's posts
revalidateTag(`user-${userId}`);
```

### Combining Strategies

```typescript
export const revalidate = 3600; // Background revalidation

// Plus on-demand for immediate updates
export async function createPost(data: FormData) {
  'use server';
  await prisma.post.create({ data });
  revalidateTag('posts');
}
```

## Summary

Combine time-based revalidation with tag-based invalidation for optimal cache control. Use tags to group related data and invalidate precisely when data changes.

## Resources

- [unstable_cache](https://nextjs.org/docs/app/api-reference/functions/unstable_cache) — Cache function documentation

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*