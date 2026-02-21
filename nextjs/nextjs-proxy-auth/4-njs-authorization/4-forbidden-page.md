---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-forbidden-page"
---

# Custom Forbidden Pages

## Introduction

When a user doesn't have permission, they shouldn't just see a generic error. A well-designed forbidden page explains what happened and gives them a path forward—whether that's requesting access, contacting support, or logging in with a different account.

## Key Concepts

**Custom error pages** in Next.js handle specific HTTP error scenarios:

- `not-found.tsx` - 404 errors
- `forbidden.tsx` - 403 errors (Next.js 15+)
- `unauthorized.tsx` - 401 errors (Next.js 15+)

## Real World Context

Good error pages:
- Reduce support tickets ("Why can't I access this?")
- Improve security by not leaking information
- Guide users to the right action
- Maintain your app's branding and UX

## Deep Dive

### Forbidden Page (Next.js 15+)

```typescript
// app/forbidden.tsx
import Link from 'next/link';

export default function Forbidden() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h1 className="text-4xl font-bold text-red-600">Access Denied</h1>
      <p className="mt-4 text-gray-600">
        You don't have permission to view this page.
      </p>
      <div className="mt-8 space-x-4">
        <Link 
          href="/dashboard" 
          className="px-4 py-2 bg-blue-500 text-white rounded"
        >
          Go to Dashboard
        </Link>
        <Link 
          href="/support" 
          className="px-4 py-2 border border-gray-300 rounded"
        >
          Request Access
        </Link>
      </div>
    </div>
  );
}
```

### Triggering the Forbidden Page

```typescript
import { forbidden } from 'next/navigation';

export default async function AdminPage() {
  const session = await getSession();
  
  if (session?.user.role !== 'admin') {
    forbidden(); // Shows app/forbidden.tsx
  }

  return <AdminDashboard />;
}
```

### Unauthorized Page (Next.js 15+)

```typescript
// app/unauthorized.tsx
import Link from 'next/link';

export default function Unauthorized() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h1 className="text-4xl font-bold">Please Sign In</h1>
      <p className="mt-4 text-gray-600">
        You need to be logged in to access this page.
      </p>
      <Link 
        href="/login" 
        className="mt-8 px-6 py-3 bg-blue-500 text-white rounded-lg"
      >
        Sign In
      </Link>
    </div>
  );
}
```

### Route-Specific Error Pages

```
app/
  admin/
    forbidden.tsx     # Admin-specific forbidden page
    page.tsx
  dashboard/
    forbidden.tsx     # Dashboard-specific forbidden page
    page.tsx
  forbidden.tsx       # Global fallback
```

### Custom Unauthorized Route

For more control, create a dedicated route:

```typescript
// app/unauthorized/page.tsx
import { headers } from 'next/headers';
import Link from 'next/link';

export default function UnauthorizedPage() {
  const headersList = await headers();
  const referer = headersList.get('referer');
  const attemptedPath = referer ? new URL(referer).pathname : null;

  return (
    <div className="max-w-md mx-auto mt-20 p-6">
      <h1 className="text-2xl font-bold text-red-600">Access Denied</h1>
      
      {attemptedPath && (
        <p className="mt-4 text-gray-600">
          You tried to access <code className="bg-gray-100 px-2 py-1 rounded">
            {attemptedPath}
          </code>
        </p>
      )}
      
      <div className="mt-6 space-y-4">
        <p className="text-gray-600">This could mean:</p>
        <ul className="list-disc list-inside space-y-2 text-gray-600">
          <li>Your account doesn't have the required permissions</li>
          <li>You need to log in with a different account</li>
          <li>This resource has restricted access</li>
        </ul>
      </div>
      
      <div className="mt-8 flex space-x-4">
        <Link href="/dashboard" className="btn-primary">
          Go to Dashboard
        </Link>
        <Link href="/login" className="btn-secondary">
          Switch Account
        </Link>
      </div>
    </div>
  );
}
```

## Common Pitfalls

1. **Leaking information**: Don't say "This admin page exists but you can't see it." Consider showing 404 for highly sensitive resources.

2. **No path forward**: Always give users something to do—go back, contact support, request access.

3. **Generic styling**: Error pages should match your app's design, not look like default browser errors.

## Best Practices

- **Be helpful, not punishing**: Explain what happened and what to do next
- **Don't leak resource existence**: For sensitive resources, 404 might be better than 403
- **Use route-specific pages**: Admin forbidden page can differ from dashboard forbidden page
- **Track forbidden attempts**: Log when users hit permission errors for security monitoring

## Summary

Custom forbidden and unauthorized pages improve UX when users lack permission. Use Next.js 15's `forbidden()` and `unauthorized()` functions to trigger them, create route-specific variants for different sections, and always provide helpful next steps. Good error pages reduce support burden and guide users appropriately.

## Resources

- [forbidden.js](https://nextjs.org/docs/app/api-reference/file-conventions/forbidden) — Custom forbidden page documentation
- [unauthorized.js](https://nextjs.org/docs/app/api-reference/file-conventions/unauthorized) — Custom unauthorized page documentation

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*