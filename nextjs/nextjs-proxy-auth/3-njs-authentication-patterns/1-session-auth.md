---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-session-auth"
---

# Session-Based Authentication

## Introduction

Session-based authentication is the classic approach to keeping users logged in. It's like a coat check at a restaurant—you hand over your coat (credentials), get a ticket (session ID), and use that ticket to retrieve your coat later.

## Key Concepts

**Session-based auth** stores user data on the **server** and uses a **cookie** to identify the session:

- **Session**: Server-side storage of user data (ID, role, preferences)
- **Session ID**: Opaque identifier stored in a cookie
- **Session Store**: Database, Redis, or memory where sessions live

## Real World Context

Session-based auth is ideal when:
- You need to revoke access instantly (security incidents, password changes)
- User data changes frequently (real-time role updates)
- You're building for browsers (cookies are automatic)
- You want simple logout (delete the session server-side)

## Deep Dive

### How It Works

1. User logs in with credentials
2. Server validates credentials and creates a session in database/Redis
3. Server sends a session ID as an **HttpOnly cookie**
4. On subsequent requests, the cookie is sent automatically
5. Server looks up the session ID to identify the user

### Session Creation

```typescript
// lib/session.ts
import { randomUUID } from 'crypto';
import { prisma } from './prisma';

export async function createSession(userId: string) {
  const sessionId = randomUUID();
  const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000); // 1 week

  await prisma.session.create({
    data: {
      id: sessionId,
      userId,
      expiresAt,
    },
  });

  return sessionId;
}

export async function getSession(sessionId: string) {
  const session = await prisma.session.findUnique({
    where: { id: sessionId },
    include: { user: true },
  });

  if (!session || session.expiresAt < new Date()) {
    return null;
  }

  return session;
}
```

### Login Route Handler

```typescript
// app/api/auth/login/route.ts
import { cookies } from 'next/headers';
import { createSession } from '@/lib/session';
import { validateCredentials } from '@/lib/auth';

export async function POST(request: Request) {
  const { email, password } = await request.json();
  const user = await validateCredentials(email, password);

  if (!user) {
    return Response.json({ error: 'Invalid credentials' }, { status: 401 });
  }

  const sessionId = await createSession(user.id);
  
  const cookieStore = await cookies();
  cookieStore.set('session', sessionId, {
    httpOnly: true,                              // Not accessible via JavaScript
    secure: process.env.NODE_ENV === 'production', // HTTPS only in prod
    sameSite: 'lax',                             // CSRF protection
    maxAge: 60 * 60 * 24 * 7,                    // 1 week
    path: '/',                                    // Available on all paths
  });

  return Response.json({ success: true });
}
```

### Validating Sessions in Proxy

```typescript
// proxy.ts
import { getSession } from '@/lib/session';

export async function proxy(request: NextRequest) {
  const sessionId = request.cookies.get('session')?.value;

  if (!sessionId) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  const session = await getSession(sessionId);
  
  if (!session) {
    // Invalid or expired session - clear the cookie
    const response = NextResponse.redirect(new URL('/login', request.url));
    response.cookies.delete('session');
    return response;
  }

  // Pass user info to routes via headers
  const response = NextResponse.next();
  response.headers.set('x-user-id', session.userId);
  return response;
}
```

### Logout

```typescript
// app/api/auth/logout/route.ts
import { cookies } from 'next/headers';
import { prisma } from '@/lib/prisma';

export async function POST() {
  const cookieStore = await cookies();
  const sessionId = cookieStore.get('session')?.value;
  
  if (sessionId) {
    // Delete session from database
    await prisma.session.delete({ where: { id: sessionId } }).catch(() => {});
    cookieStore.delete('session');
  }

  return Response.json({ success: true });
}
```

## Common Pitfalls

1. **Database lookup on every request**: Session validation queries the database each time. Use Redis or in-memory caching for better performance.

2. **Not handling expired sessions**: Always check the expiration date and clean up stale sessions.

3. **Missing cookie flags**: Forgetting `httpOnly` exposes sessions to XSS attacks.

## Best Practices

- **Use Redis for session storage**: Faster than database queries, with built-in expiration
- **Rotate session IDs after login**: Prevents session fixation attacks
- **Clear sessions on password change**: Invalidate all sessions when security-sensitive actions occur
- **Set reasonable expiration**: Balance convenience with security

## Summary

Session-based authentication stores user state server-side with only an opaque ID in the cookie. It's secure, revocable, and perfect for browser-based apps. Use `httpOnly` and `secure` flags, validate sessions on every protected request, and consider Redis for production performance.

## Resources

- [Authentication Guide](https://nextjs.org/docs/app/building-your-application/authentication) — Official authentication patterns
- [Cookies API](https://nextjs.org/docs/app/api-reference/functions/cookies) — Working with cookies in Next.js

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*