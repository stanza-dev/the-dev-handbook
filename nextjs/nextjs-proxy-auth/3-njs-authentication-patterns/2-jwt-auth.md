---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-jwt-auth"
---

# JWT (JSON Web Token) Authentication

## Introduction

Unlike sessions that require a database lookup, JWTs are self-contained passports. All the information needed to verify a user is encoded right in the token—no server-side storage required.

## Key Concepts

**JWT (JSON Web Token)** is a compact, URL-safe token format:

- **Header**: Algorithm and token type (e.g., HS256, RS256)
- **Payload**: Claims about the user (userId, email, role, exp)
- **Signature**: Cryptographic proof the token hasn't been tampered with

Format: `header.payload.signature` (Base64-encoded, dot-separated)

## Real World Context

JWTs excel when:
- You need stateless authentication (no session database)
- Your API serves multiple clients (mobile apps, SPAs, third parties)
- You want to scale horizontally without shared session storage
- Microservices need to verify users without calling the auth service

## Deep Dive

### Creating JWTs

```typescript
// lib/jwt.ts
import { SignJWT, jwtVerify } from 'jose';

const secret = new TextEncoder().encode(process.env.JWT_SECRET);

export async function createToken(userId: string, role: string) {
  return new SignJWT({ userId, role })
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('15m')  // Short-lived access token
    .sign(secret);
}

export async function verifyToken(token: string) {
  try {
    const { payload } = await jwtVerify(token, secret);
    return payload as { userId: string; role: string };
  } catch {
    return null;
  }
}
```

### Login with JWT

```typescript
// app/api/auth/login/route.ts
import { cookies } from 'next/headers';
import { createToken } from '@/lib/jwt';

export async function POST(request: Request) {
  const { email, password } = await request.json();
  const user = await validateCredentials(email, password);

  if (!user) {
    return Response.json({ error: 'Invalid credentials' }, { status: 401 });
  }

  const token = await createToken(user.id, user.role);
  
  const cookieStore = await cookies();
  cookieStore.set('token', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 60 * 15,  // 15 minutes (match token expiration)
  });

  return Response.json({ success: true });
}
```

### Verifying JWTs in Proxy

```typescript
// proxy.ts
import { jwtVerify } from 'jose';

const secret = new TextEncoder().encode(process.env.JWT_SECRET);

export async function proxy(request: NextRequest) {
  const token = request.cookies.get('token')?.value;

  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  try {
    const { payload } = await jwtVerify(token, secret);
    
    // Pass decoded user info to routes
    const response = NextResponse.next();
    response.headers.set('x-user-id', payload.userId as string);
    response.headers.set('x-user-role', payload.role as string);
    return response;
  } catch (error) {
    // Token is invalid or expired
    const response = NextResponse.redirect(new URL('/login', request.url));
    response.cookies.delete('token');
    return response;
  }
}
```

### JWT Structure Visualization

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.   // Header
eyJ1c2VySWQiOiIxMjMiLCJyb2xlIjoiYWRtaW4ifQ.  // Payload
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c   // Signature

// Decoded Payload:
{
  "userId": "123",
  "role": "admin",
  "iat": 1704067200,
  "exp": 1704068100
}
```

## Common Pitfalls

1. **Long expiration times**: JWTs can't be revoked, so long-lived tokens are a security risk. Use short expirations (15 minutes) with refresh tokens.

2. **Storing sensitive data**: The payload is only Base64-encoded, not encrypted. Anyone can decode it. Never store passwords or secrets.

3. **Using weak secrets**: Short or predictable secrets can be brute-forced. Use at least 256 bits of entropy.

## Best Practices

- **Short expiration for access tokens**: 15-30 minutes maximum
- **Use refresh tokens for extended sessions**: Store them securely and rotate on use
- **Don't store JWTs in localStorage**: Use HttpOnly cookies to prevent XSS theft
- **Include minimal claims**: Only include what's necessary; fetch fresh data from the database for sensitive operations

## Summary

JWTs are self-contained tokens that don't require server-side storage. They're verified by checking the cryptographic signature against your secret key. Use short expirations, store in HttpOnly cookies, and never include sensitive data in the payload. For revocation capabilities, combine with a token blacklist or use refresh tokens.

## Resources

- [jose Library](https://github.com/panva/jose) — Recommended JWT library for Next.js

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*