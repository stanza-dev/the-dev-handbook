---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-oauth-basics"
---

# OAuth Basics

## Introduction

Letting users sign in with Google, GitHub, or other providers isn't just convenient—it's often expected. OAuth 2.0 is the protocol that makes this possible, and understanding its flow is essential for implementing social login.

## Key Concepts

**OAuth 2.0** is an authorization protocol that allows third-party apps to access user resources without sharing passwords:

- **Authorization Server**: The provider (Google, GitHub) that authenticates users
- **Resource Server**: Where user data lives (Google's API, GitHub's API)
- **Client**: Your application
- **Authorization Code**: Temporary code exchanged for tokens
- **Access Token**: Token that grants API access

## Real World Context

OAuth is everywhere:
- "Sign in with Google" buttons
- GitHub integration in developer tools
- Social media cross-posting
- Calendar app integrations

## Deep Dive

### OAuth Authorization Code Flow

1. **User clicks "Sign in with Google"**
2. **Redirect to provider**: Your app sends user to Google with your client ID
3. **User authenticates**: Google shows login/consent screen
4. **Callback with code**: Google redirects back to your app with an authorization code
5. **Exchange code for tokens**: Your server calls Google's API with the code
6. **Get user info**: Use the access token to fetch user profile
7. **Create session**: Store user in your database, create session/JWT

### Implementation Flow

```typescript
// app/api/auth/google/route.ts - Step 2: Redirect to Google
export async function GET() {
  const params = new URLSearchParams({
    client_id: process.env.GOOGLE_CLIENT_ID!,
    redirect_uri: `${process.env.NEXT_PUBLIC_URL}/api/auth/google/callback`,
    response_type: 'code',
    scope: 'openid email profile',
    state: generateState(), // CSRF protection
  });

  return NextResponse.redirect(
    `https://accounts.google.com/o/oauth2/v2/auth?${params}`
  );
}
```

```typescript
// app/api/auth/google/callback/route.ts - Steps 5-7
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const code = searchParams.get('code');
  const state = searchParams.get('state');

  // Verify state matches what we sent (CSRF protection)
  if (!verifyState(state)) {
    return NextResponse.redirect('/login?error=invalid_state');
  }

  // Exchange code for tokens
  const tokenResponse = await fetch('https://oauth2.googleapis.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      code: code!,
      client_id: process.env.GOOGLE_CLIENT_ID!,
      client_secret: process.env.GOOGLE_CLIENT_SECRET!,
      redirect_uri: `${process.env.NEXT_PUBLIC_URL}/api/auth/google/callback`,
      grant_type: 'authorization_code',
    }),
  });

  const { access_token, id_token } = await tokenResponse.json();

  // Get user info
  const userResponse = await fetch('https://www.googleapis.com/oauth2/v2/userinfo', {
    headers: { Authorization: `Bearer ${access_token}` },
  });

  const googleUser = await userResponse.json();

  // Find or create user in database
  let user = await prisma.user.findUnique({
    where: { email: googleUser.email },
  });

  if (!user) {
    user = await prisma.user.create({
      data: {
        email: googleUser.email,
        name: googleUser.name,
        image: googleUser.picture,
        provider: 'google',
        providerId: googleUser.id,
      },
    });
  }

  // Create session and redirect
  const sessionId = await createSession(user.id);
  const cookieStore = await cookies();
  cookieStore.set('session', sessionId, { /* options */ });

  return NextResponse.redirect('/dashboard');
}
```

### State Parameter (CSRF Protection)

```typescript
// Generate state and store in cookie
function generateState(): string {
  const state = randomUUID();
  const cookieStore = await cookies();
  cookieStore.set('oauth_state', state, { httpOnly: true, maxAge: 600 });
  return state;
}

// Verify state matches
function verifyState(state: string | null): boolean {
  const cookieStore = await cookies();
  const stored = cookieStore.get('oauth_state')?.value;
  cookieStore.delete('oauth_state');
  return state === stored;
}
```

## Common Pitfalls

1. **Missing state parameter**: Without state validation, your app is vulnerable to CSRF attacks where malicious sites can link accounts.

2. **Exposing client secret**: Never send the client secret to the browser. The token exchange must happen server-side.

3. **Not handling existing users**: Check if a user with that email exists before creating duplicates.

## Best Practices

- **Use established libraries**: Auth.js (NextAuth) handles OAuth complexities for you
- **Store provider info**: Save which provider the user used to prevent account confusion
- **Handle account linking**: What happens if someone tries to OAuth with an email that exists from password login?
- **Request minimal scopes**: Only ask for permissions you actually need

## Summary

OAuth 2.0 enables "Sign in with X" functionality through a redirect-based flow. Users authenticate with the provider, your app receives an authorization code, exchanges it for tokens server-side, and uses those tokens to fetch user information. Always validate the state parameter, never expose client secrets, and consider using Auth.js to handle the complexity.

## Resources

- [OAuth 2.0 Specification](https://oauth.net/2/) — Official OAuth 2.0 documentation

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*