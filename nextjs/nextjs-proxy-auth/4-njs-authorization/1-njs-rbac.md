---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-rbac"
---

# Role-Based Access Control (RBAC)

## Introduction

Authentication answers "Who are you?" Authorization answers "What can you do?" Even authenticated users shouldn't access everything—a regular user shouldn't see admin dashboards, and a viewer shouldn't delete content.

## Key Concepts

**RBAC (Role-Based Access Control)** assigns permissions based on roles:

- **Role**: A named collection of permissions (admin, editor, viewer)
- **Permission**: A specific action on a resource (read:posts, write:users)
- **Principal**: The user or service being authorized

Simple RBAC uses roles directly. Advanced RBAC uses roles to group permissions.

## Real World Context

RBAC is essential for:
- Admin panels that only certain team members can access
- Multi-tenant apps where users only see their organization's data
- Content platforms with different creator/consumer permissions
- Enterprise apps with complex organizational hierarchies

## Deep Dive

### Common Role Patterns

```typescript
// Simple role hierarchy
const ROLES = {
  ADMIN: 'admin',      // Full access
  EDITOR: 'editor',    // Create/edit content
  VIEWER: 'viewer',    // Read-only access
} as const;

// Permission-based roles
const PERMISSIONS = {
  admin: ['read', 'write', 'delete', 'manage_users'],
  editor: ['read', 'write'],
  viewer: ['read'],
};

function hasPermission(role: string, permission: string): boolean {
  return PERMISSIONS[role]?.includes(permission) ?? false;
}
```

### Implementing RBAC in Proxy

```typescript
import { jwtVerify } from 'jose';

const secret = new TextEncoder().encode(process.env.JWT_SECRET);

// Define route-role mappings
const ROUTE_PERMISSIONS: Record<string, string[]> = {
  '/admin': ['admin'],
  '/dashboard/settings': ['admin', 'editor'],
  '/dashboard': ['admin', 'editor', 'viewer'],
};

function getRequiredRoles(pathname: string): string[] | null {
  for (const [route, roles] of Object.entries(ROUTE_PERMISSIONS)) {
    if (pathname.startsWith(route)) {
      return roles;
    }
  }
  return null; // Public route
}

export async function proxy(request: NextRequest) {
  const token = request.cookies.get('token')?.value;
  const { pathname } = request.nextUrl;

  const requiredRoles = getRequiredRoles(pathname);
  
  // Public route, no auth needed
  if (!requiredRoles) {
    return NextResponse.next();
  }

  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  try {
    const { payload } = await jwtVerify(token, secret);
    const userRole = payload.role as string;

    // Check if user's role is allowed
    if (!requiredRoles.includes(userRole)) {
      return NextResponse.redirect(new URL('/unauthorized', request.url));
    }

    // Pass role to downstream handlers
    const response = NextResponse.next();
    response.headers.set('x-user-role', userRole);
    return response;
  } catch {
    return NextResponse.redirect(new URL('/login', request.url));
  }
}
```

### Role Hierarchy

Admins can do everything editors can, editors can do everything viewers can:

```typescript
const ROLE_HIERARCHY: Record<string, number> = {
  admin: 3,
  editor: 2,
  viewer: 1,
};

function hasMinimumRole(userRole: string, requiredRole: string): boolean {
  return (ROLE_HIERARCHY[userRole] ?? 0) >= (ROLE_HIERARCHY[requiredRole] ?? 0);
}

// Usage: hasMinimumRole('admin', 'editor') → true
// Usage: hasMinimumRole('viewer', 'editor') → false
```

## Common Pitfalls

1. **Hardcoding roles in multiple places**: Role checks scattered throughout the codebase become unmaintainable. Centralize role definitions.

2. **Only checking in proxy**: Proxy is the first line of defense, but verify permissions again in route handlers for defense in depth.

3. **Not handling role changes**: If a user's role changes, their existing token/session still has the old role until it expires.

## Best Practices

- **Centralize role definitions**: Keep all roles and permissions in a single file
- **Use role hierarchy when appropriate**: Reduces duplication in permission checks
- **Defense in depth**: Check permissions in proxy AND in route handlers
- **Log authorization failures**: Track when users attempt unauthorized access

## Summary

RBAC controls what authenticated users can do based on their assigned role. Define roles centrally, map routes to required roles, and check permissions in proxy. Use role hierarchy for cleaner code, and always verify permissions at multiple levels for security.

## Resources

- [Authorization Patterns](https://nextjs.org/docs/app/building-your-application/authentication#authorization) — Next.js authorization documentation

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*