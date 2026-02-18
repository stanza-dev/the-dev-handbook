---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-server-component-auth"
---

# Authorization in Server Components

## Introduction

Proxy is your first line of defense, but sometimes you need authorization checks right where the data is fetched. Server Components give you the power to verify permissions at the component level—before any sensitive data is even loaded.

## Key Concepts

**Component-level authorization** checks permissions within the component itself:

- **Early return**: Redirect before rendering sensitive content
- **Conditional fetching**: Only query data the user can access
- **Conditional rendering**: Show different UI based on roles

## Real World Context

Component-level auth is valuable when:
- Different parts of a page have different permission requirements
- You need to fetch user-specific data based on their role
- A single page serves multiple roles with different views
- You want defense in depth beyond proxy

## Deep Dive

### Basic Authorization in Server Components

```typescript
// app/admin/page.tsx
import { cookies } from 'next/headers';
import { verifySession } from '@/lib/auth';
import { redirect } from 'next/navigation';

export default async function AdminPage() {
  const session = await verifySession(cookies().get('session')?.value);

  // Not logged in
  if (!session) {
    redirect('/login');
  }

  // Logged in but wrong role
  if (session.user.role !== 'admin') {
    redirect('/unauthorized');
  }

  // Fetch admin-only data
  const stats = await fetchAdminStats();

  return (
    <div>
      <h1>Admin Dashboard</h1>
      <p>Welcome, {session.user.name}!</p>
      <AdminStats data={stats} />
    </div>
  );
}
```

### Creating a Reusable Auth Helper

```typescript
// lib/auth.ts
import { cookies } from 'next/headers';
import { redirect } from 'next/navigation';
import { cache } from 'react';

// Cache the session check per request
export const getSession = cache(async () => {
  const sessionId = cookies().get('session')?.value;
  if (!sessionId) return null;
  
  return await verifySession(sessionId);
});

export async function requireAuth() {
  const session = await getSession();
  if (!session) {
    redirect('/login');
  }
  return session;
}

export async function requireRole(allowedRoles: string[]) {
  const session = await requireAuth();
  if (!allowedRoles.includes(session.user.role)) {
    redirect('/unauthorized');
  }
  return session;
}
```

### Using Auth Helpers

```typescript
// app/admin/page.tsx
import { requireRole } from '@/lib/auth';

export default async function AdminPage() {
  const session = await requireRole(['admin']);
  
  return <h1>Welcome, {session.user.name}!</h1>;
}

// app/dashboard/page.tsx
import { requireRole } from '@/lib/auth';

export default async function DashboardPage() {
  const session = await requireRole(['admin', 'editor', 'viewer']);
  
  return <h1>Dashboard for {session.user.role}</h1>;
}
```

### Conditional Rendering Based on Role

```typescript
import { getSession } from '@/lib/auth';

export default async function DashboardPage() {
  const session = await getSession();
  
  if (!session) {
    redirect('/login');
  }

  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Everyone sees this */}
      <UserStats userId={session.user.id} />
      
      {/* Only editors and admins */}
      {['editor', 'admin'].includes(session.user.role) && (
        <ContentEditor />
      )}
      
      {/* Only admins */}
      {session.user.role === 'admin' && (
        <AdminPanel />
      )}
    </div>
  );
}
```

### Reading User Info from Proxy Headers

If proxy already verified the user and set headers:

```typescript
import { headers } from 'next/headers';

export default async function ProtectedPage() {
  const headersList = headers();
  const userId = headersList.get('x-user-id');
  const userRole = headersList.get('x-user-role');

  if (!userId) {
    redirect('/login');
  }

  // No need to verify again - proxy already did
  const userData = await fetchUserData(userId);

  return <UserDashboard data={userData} role={userRole} />;
}
```

## Common Pitfalls

1. **Redundant session lookups**: Without caching, each component checks the session independently. Use React's `cache()` to dedupe.

2. **Relying only on client checks**: Never trust client-side role checks. Always verify server-side.

3. **Forgetting to await redirects**: `redirect()` throws an error to halt rendering, but forgetting `return` can cause issues.

## Best Practices

- **Use `cache()` for session checks**: Prevents multiple database hits per request
- **Create reusable auth helpers**: `requireAuth()` and `requireRole()` functions
- **Defense in depth**: Check in proxy AND components for sensitive routes
- **Keep UI feedback graceful**: Use loading states while auth checks run

## Summary

Server Component authorization provides granular, component-level permission checks. Use cached session helpers to avoid redundant lookups, create reusable `requireRole()` functions, and combine with proxy for defense in depth. Conditional rendering lets you show different UI to different roles on the same page.

## Resources

- [Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components) — Understanding Server Components

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*