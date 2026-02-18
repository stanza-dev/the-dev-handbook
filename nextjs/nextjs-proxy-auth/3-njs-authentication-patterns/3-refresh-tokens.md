---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-refresh-tokens"
---

# Refresh Token Pattern

## Introduction

Short-lived access tokens are secure but annoying—users would need to log in every 15 minutes. Refresh tokens solve this by enabling silent token renewal without re-entering credentials.

## Key Concepts

**Two-token system**:
- **Access Token**: Short-lived (15 minutes), used for API requests
- **Refresh Token**: Long-lived (7 days), used only to get new access tokens

Why separate them?
- Access tokens are used frequently and exposed to more attack surfaces
- Refresh tokens are used rarely and can be stored more securely

## Real World Context

The refresh pattern is essential for:
- Mobile apps that need to stay logged in
- SPAs that make frequent API calls
- Any app balancing security (short tokens) with UX (staying logged in)

## Deep Dive

### Token Creation

```typescript
// lib/tokens.ts
import { SignJWT, jwtVerify } from 'jose';
import { randomUUID } from 'crypto';

const accessSecret = new TextEncoder().encode(process.env.ACCESS_TOKEN_SECRET);
const refreshSecret = new TextEncoder().encode(process.env.REFRESH_TOKEN_SECRET);

export async function createTokens(userId: string, role: string) {
  const accessToken = await new SignJWT({ userId, role })
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('15m')
    .sign(accessSecret);

  const refreshToken = await new SignJWT({ userId, tokenId: randomUUID() })
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('7d')
    .sign(refreshSecret);

  return { accessToken, refreshToken };
}
```

### Login: Issue Both Tokens

```typescript
// app/api/auth/login/route.ts
export async function POST(request: Request) {
  const { email, password } = await request.json();
  const user = await validateCredentials(email, password);

  if (!user) {
    return Response.json({ error: 'Invalid credentials' }, { status: 401 });
  }

  const { accessToken, refreshToken } = await createTokens(user.id, user.role);
  
  // Store refresh token in database for revocation capability
  await prisma.refreshToken.create({
    data: { token: refreshToken, userId: user.id },
  });

  const cookieOptions = {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax' as const,
    path: '/',
  };

  cookies().set('access_token', accessToken, {
    ...cookieOptions,
    maxAge: 60 * 15,  // 15 minutes
  });

  cookies().set('refresh_token', refreshToken, {
    ...cookieOptions,
    maxAge: 60 * 60 * 24 * 7,  // 7 days
  });

  return Response.json({ success: true });
}
```

### Refresh Endpoint

```typescript
// app/api/auth/refresh/route.ts
import { jwtVerify } from 'jose';
import { createTokens } from '@/lib/tokens';

export async function POST(request: Request) {
  const refreshToken = cookies().get('refresh_token')?.value;

  if (!refreshToken) {
    return Response.json({ error: 'No refresh token' }, { status: 401 });
  }

  try {
    // Verify refresh token
    const { payload } = await jwtVerify(refreshToken, refreshSecret);
    
    // Check if token is in database (not revoked)
    const storedToken = await prisma.refreshToken.findUnique({
      where: { token: refreshToken },
      include: { user: true },
    });

    if (!storedToken) {
      return Response.json({ error: 'Invalid refresh token' }, { status: 401 });
    }

    // Rotate: Delete old, create new
    await prisma.refreshToken.delete({ where: { token: refreshToken } });
    
    const { accessToken, refreshToken: newRefreshToken } = 
      await createTokens(storedToken.userId, storedToken.user.role);

    await prisma.refreshToken.create({
      data: { token: newRefreshToken, userId: storedToken.userId },
    });

    // Set new cookies
    cookies().set('access_token', accessToken, { /* ... */ });
    cookies().set('refresh_token', newRefreshToken, { /* ... */ });

    return Response.json({ success: true });
  } catch {
    return Response.json({ error: 'Invalid token' }, { status: 401 });
  }
}
```

### Proxy with Auto-Refresh

```typescript
// proxy.ts
export async function proxy(request: NextRequest) {
  const accessToken = request.cookies.get('access_token')?.value;
  const refreshToken = request.cookies.get('refresh_token')?.value;

  if (accessToken) {
    try {
      const { payload } = await jwtVerify(accessToken, accessSecret);
      const response = NextResponse.next();
      response.headers.set('x-user-id', payload.userId as string);
      return response;
    } catch {
      // Access token expired, try refresh
    }
  }

  // No valid access token - redirect to login
  // (Client-side will handle refresh via API call)
  if (!refreshToken) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Has refresh token but no valid access token
  // Could auto-refresh here or let client handle it
  return NextResponse.redirect(new URL('/auth/refresh', request.url));
}
```

## Common Pitfalls

1. **Not rotating refresh tokens**: Reusing the same refresh token forever means a stolen token has unlimited lifetime. Always issue a new one.

2. **Storing refresh tokens client-side**: Refresh tokens in localStorage can be stolen via XSS. Use HttpOnly cookies.

3. **Not storing refresh tokens server-side**: Without server storage, you can't revoke tokens on logout or security events.

## Best Practices

- **Rotate refresh tokens on each use**: Delete the old one, issue a new one
- **Store refresh tokens in the database**: Enables revocation and audit trails
- **Use different secrets for access and refresh tokens**: Compromising one doesn't compromise the other
- **Clear all tokens on password change**: Revoke all refresh tokens when security-sensitive actions occur

## Summary

The refresh token pattern combines short-lived access tokens (security) with long-lived refresh tokens (convenience). Store refresh tokens server-side for revocation capability, rotate them on each use, and use HttpOnly cookies for both. This pattern gives you the best of both JWTs and sessions.

## Resources

- [Token Refresh Architecture](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/) — In-depth guide to refresh token patterns

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*