---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-nextauth-callbacks"
---

# Auth.js Callbacks

## Introduction

Callbacks are hooks into the authentication flow. They let you customize what happens when users sign in, when sessions are created, and what data gets stored in tokens.

## Key Concepts

**Callbacks** are functions called at specific points in the auth flow:

- `signIn`: Called when a user attempts to sign in
- `redirect`: Called when redirecting after sign in/out
- `jwt`: Called when a JWT is created or updated
- `session`: Called when session is checked

## Real World Context

Callbacks enable:
- Restricting sign-ups to specific email domains
- Adding custom data to sessions (role, organization)
- Logging authentication events
- Blocking banned users

## Deep Dive

### The signIn Callback

Control who can sign in:

```typescript
export const { handlers, auth } = NextAuth({
  callbacks: {
    async signIn({ user, account, profile }) {
      // Only allow company emails
      if (user.email?.endsWith('@company.com')) {
        return true;
      }
      
      // Block sign-in
      return false;
      
      // Or redirect to an error page
      return '/auth/error?error=AccessDenied';
    },
  },
});
```

### The jwt Callback

Customize the JWT token:

```typescript
callbacks: {
  async jwt({ token, user, account, trigger }) {
    // Initial sign in
    if (user) {
      token.id = user.id;
      token.role = user.role || 'user';
    }

    // Update session from client (useSession().update())
    if (trigger === 'update') {
      const freshUser = await getUserById(token.id as string);
      token.role = freshUser.role;
    }

    return token;
  },
}
```

### The session Callback

Customize what's available in the session:

```typescript
callbacks: {
  async session({ session, token }) {
    // Expose token data to the session
    if (token) {
      session.user.id = token.id as string;
      session.user.role = token.role as string;
    }
    return session;
  },
}
```

### The redirect Callback

Control post-auth redirects:

```typescript
callbacks: {
  async redirect({ url, baseUrl }) {
    // Redirect to the URL they were trying to access
    if (url.startsWith('/')) return `${baseUrl}${url}`;
    
    // Allow redirects to same origin
    if (new URL(url).origin === baseUrl) return url;
    
    // Default to home
    return baseUrl;
  },
}
```

### Complete Example with All Callbacks

```typescript
// auth.ts
import NextAuth from 'next-auth';
import GitHub from 'next-auth/providers/github';
import { prisma } from '@/lib/prisma';

export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [GitHub],
  callbacks: {
    async signIn({ user, account }) {
      // Check if user is banned
      const dbUser = await prisma.user.findUnique({
        where: { email: user.email! },
      });
      
      if (dbUser?.banned) {
        return '/auth/banned';
      }

      // Log sign-in
      await prisma.authLog.create({
        data: {
          email: user.email!,
          provider: account?.provider,
          action: 'sign_in',
        },
      });

      return true;
    },

    async jwt({ token, user }) {
      if (user) {
        // First sign in - fetch or create user
        const dbUser = await prisma.user.upsert({
          where: { email: user.email! },
          update: { lastLogin: new Date() },
          create: {
            email: user.email!,
            name: user.name,
            image: user.image,
            role: 'user',
          },
        });

        token.id = dbUser.id;
        token.role = dbUser.role;
      }
      return token;
    },

    async session({ session, token }) {
      session.user.id = token.id as string;
      session.user.role = token.role as string;
      return session;
    },
  },
});
```

### TypeScript Types

Extend the types to include custom fields:

```typescript
// types/next-auth.d.ts
import { DefaultSession } from 'next-auth';

declare module 'next-auth' {
  interface Session {
    user: {
      id: string;
      role: string;
    } & DefaultSession['user'];
  }
}
```

## Common Pitfalls

1. **Async operations in callbacks**: Keep them fast—they run on every request. Cache or optimize database calls.

2. **Forgetting to return**: All callbacks must return a value. `signIn` returns boolean/string, others return their modified parameter.

3. **jwt vs session confusion**: `jwt` runs first and populates the token. `session` runs second and populates the session from the token.

## Best Practices

- **Keep callbacks lean**: Heavy operations slow down every auth check
- **Use database sessions for heavy data**: If you need lots of user data, use database sessions instead of JWTs
- **Type your extensions**: Add TypeScript declarations for custom session fields
- **Log important events**: Track sign-ins, failures, and suspicious activity

## Summary

Callbacks customize the Auth.js flow: `signIn` controls access, `jwt` customizes the token, `session` shapes what's exposed to your app, and `redirect` controls navigation. Use them to add custom data, restrict access, and log authentication events. Keep them fast and always return the expected value.

## Resources

- [Auth.js Callbacks](https://authjs.dev/guides/basics/callbacks) — Callback reference documentation

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*