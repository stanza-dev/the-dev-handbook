---
source_course: "react-server-components"
source_lesson: "react-server-components-action-security"
---

# Security Best Practices for Server Actions

Server Actions are public HTTP endpoints. Treat them with the same security considerations as API routes.

## Authentication & Authorization

Always verify the user has permission:

```tsx
'use server';

import { auth } from './auth';
import { redirect } from 'next/navigation';

export async function deletePost(postId: string) {
  // 1. Authenticate
  const session = await auth();
  if (!session) {
    redirect('/login');
  }
  
  // 2. Authorize
  const post = await db.posts.findUnique({ where: { id: postId } });
  if (post.authorId !== session.user.id) {
    throw new Error('Not authorized');
  }
  
  // 3. Perform action
  await db.posts.delete({ where: { id: postId } });
  revalidatePath('/posts');
}
```

## Input Validation

Never trust client input:

```tsx
'use server';

import { z } from 'zod';

const CreatePostSchema = z.object({
  title: z.string().min(1).max(100),
  content: z.string().min(1).max(10000),
  categoryId: z.string().uuid()
});

export async function createPost(formData: FormData) {
  // Validate all input
  const result = CreatePostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
    categoryId: formData.get('categoryId')
  });
  
  if (!result.success) {
    return { error: result.error.flatten() };
  }
  
  const { title, content, categoryId } = result.data;
  
  // Verify categoryId exists (don't trust client)
  const category = await db.categories.findUnique({
    where: { id: categoryId }
  });
  
  if (!category) {
    return { error: 'Invalid category' };
  }
  
  await db.posts.create({ data: { title, content, categoryId } });
}
```

## Closure Security

Be careful with closures - they serialize values:

```tsx
// ⚠️ Dangerous - userId could be tampered with
async function UserProfile({ userId }) {
  async function deleteAccount() {
    'use server';
    // userId is serialized and sent to client!
    await db.users.delete({ where: { id: userId } });
  }
  
  return <button formAction={deleteAccount}>Delete</button>;
}

// ✅ Safe - verify on server
async function UserProfile() {
  async function deleteAccount() {
    'use server';
    const session = await auth();
    if (!session) throw new Error('Not authenticated');
    
    // Use authenticated user's ID, not a closure value
    await db.users.delete({ where: { id: session.user.id } });
  }
  
  return <button formAction={deleteAccount}>Delete</button>;
}
```

## Rate Limiting

Protect against abuse:

```tsx
'use server';

import { rateLimit } from './rate-limit';
import { headers } from 'next/headers';

export async function submitForm(formData: FormData) {
  const ip = headers().get('x-forwarded-for') ?? 'unknown';
  
  const { success } = await rateLimit.limit(ip);
  if (!success) {
    return { error: 'Too many requests. Please try again later.' };
  }
  
  // Process form...
}
```

## CSRF Protection

Next.js automatically includes CSRF protection for Server Actions. The action can only be called from your own site.

## Sensitive Data

Never return sensitive data:

```tsx
'use server';

// ❌ Bad - returns sensitive data
export async function getUser(id: string) {
  return await db.users.findUnique({ where: { id } });
  // Might include password hash, internal IDs, etc.
}

// ✅ Good - select only needed fields
export async function getUser(id: string) {
  return await db.users.findUnique({
    where: { id },
    select: {
      id: true,
      name: true,
      email: true,
      avatar: true
    }
  });
}
```

## Resources

- [Server Actions Security](https://react.dev/reference/rsc/server-functions#security) — Security considerations for Server Functions

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*