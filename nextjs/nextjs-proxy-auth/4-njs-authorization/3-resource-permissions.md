---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-resource-permissions"
---

# Resource-Level Permissions

## Introduction

Role-based checks answer "Can this user access admin pages?" But what about "Can this user edit THIS specific post?" Resource-level permissions check ownership and access rights on individual items.

## Key Concepts

**Resource-level authorization** validates access to specific resources:

- **Ownership**: Does the user own this resource?
- **Organization**: Does the resource belong to the user's organization?
- **Sharing**: Has the resource been shared with this user?
- **Capability**: Does the user have the required permission for this action?

## Real World Context

Resource-level permissions are everywhere:
- Google Docs: Who can view/edit THIS document?
- GitHub: Who can push to THIS repository?
- Notion: Who can access THIS workspace?
- Slack: Who can post in THIS channel?

## Deep Dive

### Ownership Check

```typescript
// app/posts/[id]/edit/page.tsx
import { requireAuth } from '@/lib/auth';
import { notFound, redirect } from 'next/navigation';

export default async function EditPostPage({ 
  params 
}: { 
  params: { id: string } 
}) {
  const session = await requireAuth();
  
  const post = await prisma.post.findUnique({
    where: { id: params.id },
  });

  if (!post) {
    notFound();
  }

  // Check ownership (unless admin)
  if (post.authorId !== session.user.id && session.user.role !== 'admin') {
    redirect('/unauthorized');
  }

  return <PostEditor post={post} />;
}
```

### Organization-Based Access

```typescript
// Multi-tenant pattern
async function canAccessResource(
  userId: string,
  resourceId: string
): Promise<boolean> {
  const user = await prisma.user.findUnique({
    where: { id: userId },
    include: { organization: true },
  });

  const resource = await prisma.resource.findUnique({
    where: { id: resourceId },
  });

  // Same organization?
  return user?.organizationId === resource?.organizationId;
}

// Usage in Server Component
export default async function ResourcePage({ params }) {
  const session = await requireAuth();
  
  const canAccess = await canAccessResource(session.user.id, params.id);
  
  if (!canAccess) {
    redirect('/unauthorized');
  }

  const resource = await fetchResource(params.id);
  return <ResourceView data={resource} />;
}
```

### Sharing-Based Access

```typescript
// Check if resource is shared with user
async function hasAccess(
  userId: string,
  resourceId: string,
  permission: 'view' | 'edit' | 'admin'
): Promise<boolean> {
  // Check direct share
  const share = await prisma.resourceShare.findFirst({
    where: {
      resourceId,
      userId,
      permission: { in: getPermissionsAtOrAbove(permission) },
    },
  });

  if (share) return true;

  // Check if public
  const resource = await prisma.resource.findUnique({
    where: { id: resourceId },
  });

  if (resource?.isPublic && permission === 'view') {
    return true;
  }

  // Check ownership
  return resource?.ownerId === userId;
}

function getPermissionsAtOrAbove(permission: string): string[] {
  const hierarchy = ['view', 'edit', 'admin'];
  const index = hierarchy.indexOf(permission);
  return hierarchy.slice(index);
}
```

### In Route Handlers

```typescript
// app/api/posts/[id]/route.ts
import { NextRequest } from 'next/server';

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const userId = request.headers.get('x-user-id');
  
  if (!userId) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const post = await prisma.post.findUnique({
    where: { id: params.id },
  });

  if (!post) {
    return Response.json({ error: 'Not found' }, { status: 404 });
  }

  // Only owner can delete
  if (post.authorId !== userId) {
    return Response.json({ error: 'Forbidden' }, { status: 403 });
  }

  await prisma.post.delete({ where: { id: params.id } });
  return new Response(null, { status: 204 });
}
```

## Common Pitfalls

1. **Checking role but not ownership**: An editor role doesn't mean they can edit EVERY resource—only their own.

2. **N+1 permission checks**: Fetching resources then checking permissions one by one. Join the permission check with the initial query.

3. **Forgetting admin overrides**: Admins often need to access any resource for support/moderation.

## Best Practices

- **Check at the data layer**: Add permission checks to your data access functions
- **Use database joins**: Combine resource fetch with permission check in one query
- **Create policy functions**: `canEdit(user, post)`, `canDelete(user, comment)`
- **Return 404 vs 403 strategically**: Sometimes you don't want to reveal that a resource exists

## Summary

Resource-level permissions go beyond role checks to validate access to specific items. Check ownership, organization membership, and explicit shares. Always verify permissions before any create, update, or delete operation. Combine permission checks with data fetching for efficiency, and create reusable policy functions for consistency.

## Resources

- [Data Security](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#data-security) — Securing data in Next.js

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*