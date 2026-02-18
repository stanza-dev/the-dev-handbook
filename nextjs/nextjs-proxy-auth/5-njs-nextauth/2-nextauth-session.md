---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-nextauth-session"
---

# Accessing the Session

## Introduction

Once a user is signed in, you need to access their session data—name, email, image, role—throughout your app. Auth.js provides different APIs for Server Components, Client Components, and Route Handlers.

## Key Concepts

**Session access patterns**:

- `auth()` - Server Components, Route Handlers, Server Actions
- `useSession()` - Client Components (requires SessionProvider)
- `getSession()` - Deprecated, use `auth()` instead

## Real World Context

You'll access sessions to:
- Display user info in the header
- Conditionally render UI based on auth status
- Include user ID in database queries
- Protect pages and API routes

## Deep Dive

### Server Components (Recommended)

```typescript
// app/dashboard/page.tsx
import { auth } from '@/auth';
import { redirect } from 'next/navigation';

export default async function Dashboard() {
  const session = await auth();

  if (!session?.user) {
    redirect('/api/auth/signin');
  }

  return (
    <div>
      <h1>Welcome, {session.user.name}!</h1>
      <p>Email: {session.user.email}</p>
      <img 
        src={session.user.image || '/default-avatar.png'} 
        alt="Profile" 
        className="w-16 h-16 rounded-full"
      />
    </div>
  );
}
```

### Client Components

First, wrap your app in `SessionProvider`:

```typescript
// app/layout.tsx
import { SessionProvider } from 'next-auth/react';

export default function RootLayout({ 
  children 
}: { 
  children: React.ReactNode 
}) {
  return (
    <html>
      <body>
        <SessionProvider>{children}</SessionProvider>
      </body>
    </html>
  );
}
```

Then use the `useSession` hook:

```typescript
// components/UserMenu.tsx
'use client';
import { useSession, signIn, signOut } from 'next-auth/react';

export function UserMenu() {
  const { data: session, status } = useSession();

  if (status === 'loading') {
    return <div className="animate-pulse">Loading...</div>;
  }

  if (!session) {
    return (
      <button onClick={() => signIn()}>
        Sign In
      </button>
    );
  }

  return (
    <div className="flex items-center gap-4">
      <span>Hi, {session.user?.name}</span>
      <button onClick={() => signOut()}>
        Sign Out
      </button>
    </div>
  );
}
```

### Route Handlers

```typescript
// app/api/user/profile/route.ts
import { auth } from '@/auth';

export async function GET() {
  const session = await auth();

  if (!session?.user) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const profile = await fetchUserProfile(session.user.email!);
  return Response.json(profile);
}
```

### Session Data Structure

```typescript
type Session = {
  user?: {
    name?: string | null;
    email?: string | null;
    image?: string | null;
  };
  expires: string; // ISO date string
};
```

### Extending the Session (Adding Custom Data)

```typescript
// auth.ts
export const { handlers, auth } = NextAuth({
  // ... providers
  callbacks: {
    session({ session, token }) {
      // Add custom fields to session
      if (token.sub) {
        session.user.id = token.sub;
        session.user.role = token.role as string;
      }
      return session;
    },
    jwt({ token, user }) {
      // Add custom fields to token
      if (user) {
        token.role = user.role;
      }
      return token;
    },
  },
});
```

## Common Pitfalls

1. **Missing SessionProvider**: `useSession()` returns undefined without the provider wrapper.

2. **Not handling loading state**: `status` can be 'loading', 'authenticated', or 'unauthenticated'. Always handle loading.

3. **Assuming session exists**: Always check `session?.user` with optional chaining.

## Best Practices

- **Prefer Server Components**: Use `auth()` for most cases—it's faster and doesn't require client-side JS
- **Handle all states**: Loading, authenticated, and unauthenticated
- **Extend session via callbacks**: Don't refetch user data—add what you need to the session
- **Cache session checks**: Use React's `cache()` if calling `auth()` multiple times per request

## Summary

Access sessions with `auth()` in Server Components and Route Handlers, and `useSession()` in Client Components (with SessionProvider). Always handle loading and unauthenticated states. Extend the session with callbacks to include custom user data like roles and IDs.

## Resources

- [Session Management](https://authjs.dev/getting-started/session-management) — Auth.js session documentation

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*