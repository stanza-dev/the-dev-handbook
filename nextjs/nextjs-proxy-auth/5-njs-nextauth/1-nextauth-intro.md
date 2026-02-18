---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-nextauth-intro"
---

# Introduction to Auth.js

## Introduction

Building authentication from scratch is complex—OAuth flows, session management, CSRF protection, secure cookies. Auth.js (formerly NextAuth.js) handles all of this for you, with first-class support for Next.js.

## Key Concepts

**Auth.js** is a complete authentication solution:

- **Providers**: OAuth (Google, GitHub, etc.), Credentials (email/password), Email magic links
- **Session handling**: JWT or database sessions, automatic refresh
- **Callbacks**: Hook into the auth flow for customization
- **Adapters**: Store users in any database (Prisma, Drizzle, etc.)

## Real World Context

Auth.js powers thousands of production apps because:
- OAuth is handled correctly (state, PKCE, token refresh)
- Security best practices are built-in
- Adding new providers takes minutes, not hours
- Session management just works

## Deep Dive

### Installation

```bash
npm install next-auth@beta
```

### Core Configuration

Create `auth.ts` in your project root:

```typescript
// auth.ts
import NextAuth from 'next-auth';
import GitHub from 'next-auth/providers/github';
import Google from 'next-auth/providers/google';

export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [
    GitHub({
      clientId: process.env.AUTH_GITHUB_ID,
      clientSecret: process.env.AUTH_GITHUB_SECRET,
    }),
    Google({
      clientId: process.env.AUTH_GOOGLE_ID,
      clientSecret: process.env.AUTH_GOOGLE_SECRET,
    }),
  ],
});
```

### Route Handler Setup

Create `app/api/auth/[...nextauth]/route.ts`:

```typescript
import { handlers } from '@/auth';
export const { GET, POST } = handlers;
```

This single file creates all necessary endpoints:
- `GET /api/auth/signin` - Sign-in page
- `GET /api/auth/signout` - Sign-out page
- `GET /api/auth/session` - Get current session
- `GET /api/auth/csrf` - CSRF token
- `GET /api/auth/providers` - List of providers
- `POST /api/auth/signin/:provider` - Start OAuth flow
- `GET /api/auth/callback/:provider` - OAuth callback

### Environment Variables

```bash
# .env.local
AUTH_SECRET=your-random-secret-min-32-chars
AUTH_GITHUB_ID=your-github-client-id
AUTH_GITHUB_SECRET=your-github-client-secret
AUTH_GOOGLE_ID=your-google-client-id
AUTH_GOOGLE_SECRET=your-google-client-secret
```

Generate `AUTH_SECRET`:
```bash
npx auth secret
```

### Exports Overview

```typescript
export const { 
  handlers,  // GET and POST for route handler
  auth,      // Get session (Server Components, API routes)
  signIn,    // Programmatic sign-in
  signOut,   // Programmatic sign-out
} = NextAuth({ /* config */ });
```

## Common Pitfalls

1. **Missing AUTH_SECRET**: Auth.js won't work without a secret. Generate one and add it to your environment.

2. **Wrong callback URLs**: OAuth providers need exact callback URLs. Use `http://localhost:3000/api/auth/callback/github` for local development.

3. **Forgetting the route handler**: The `[...nextauth]/route.ts` file must exist for Auth.js to function.

## Best Practices

- **Use environment variable prefixes**: `AUTH_GITHUB_ID` instead of `GITHUB_ID` for clarity
- **Generate a strong secret**: Use `npx auth secret` to generate a cryptographically secure secret
- **Start with one provider**: Get one working before adding more
- **Check the console**: Auth.js logs helpful debugging information

## Summary

Auth.js provides production-ready authentication for Next.js with minimal setup. Install the package, create an auth config with your providers, set up the route handler, and you have OAuth, sessions, and security handled. The library exports `auth()`, `signIn()`, and `signOut()` for use throughout your application.

## Resources

- [Auth.js Documentation](https://authjs.dev/getting-started/installation?framework=Next.js) — Official Auth.js setup guide
- [Auth.js Providers](https://authjs.dev/getting-started/providers) — Available authentication providers

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*