---
source_course: "nestjs-security"
source_lesson: "nestjs-security-csrf"
---

# CSRF Protection

## Introduction

Cross-Site Request Forgery (CSRF) tricks authenticated users into performing unintended actions. While less relevant for pure API backends (token-based auth), applications using cookies for authentication must implement CSRF protection.

## Key Concepts

- **CSRF**: Attack that tricks users into submitting malicious requests
- **CSRF Token**: Unpredictable value that validates request origin
- **SameSite Cookie**: Cookie attribute that helps prevent CSRF
- **Double Submit Cookie**: CSRF mitigation pattern

## Real World Context

Imagine a user is logged into their bank. They visit a malicious site that submits a hidden form to transfer money. Without CSRF protection, the bank's server processes the request because the user's session cookie is automatically included.

## Deep Dive

### Cookie-Based CSRF Protection (Express)

> **Note:** The `csurf` package is deprecated and no longer maintained. The NestJS docs now recommend `csrf-csrf` for Express or `@fastify/csrf-protection` for Fastify.

Install the `csrf-csrf` package alongside `cookie-parser` for the double-submit cookie pattern.

```bash
npm install csrf-csrf cookie-parser
```

Configure the CSRF middleware with a secret and secure cookie options. The double-submit pattern compares a cookie value against a request header.

```typescript
import * as cookieParser from 'cookie-parser';
import { doubleCsrf } from 'csrf-csrf';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.use(cookieParser());
  
  const { doubleCsrfProtection, generateToken } = doubleCsrf({
    getSecret: () => process.env.CSRF_SECRET,
    cookieName: '__csrf',
    cookieOptions: { sameSite: 'strict', secure: true },
  });
  app.use(doubleCsrfProtection);
  
  await app.listen(3000);
}
```

The `sameSite: 'strict'` and `secure: true` options ensure the CSRF cookie is only sent over HTTPS and never in cross-site requests.

### Fastify CSRF Protection

If you use Fastify instead of Express, install the official Fastify CSRF plugin.

```bash
npm install @fastify/csrf-protection
```

The Fastify plugin provides equivalent protection with Fastify-native session and cookie handling.

### Getting CSRF Token

With `csrf-csrf`, expose an endpoint that generates and returns the CSRF token to your frontend. The frontend must include this token in subsequent state-changing requests.

```typescript
import { doubleCsrf } from 'csrf-csrf';
const { generateToken } = doubleCsrf(doubleCsrfOptions);

@Controller()
export class AppController {
  @Get('csrf-token')
  getCsrfToken(@Req() req, @Res({ passthrough: true }) res): { csrfToken: string } {
    const token = generateToken(req, res);
    return { csrfToken: token };
  }
}
```

The frontend should call this endpoint on page load and attach the token as a header (e.g., `X-CSRF-Token`) in POST/PUT/DELETE requests.

### SameSite Cookies (Modern Approach)

Modern browsers support the `SameSite` cookie attribute, which provides built-in CSRF protection by restricting when cookies are sent cross-site.

```typescript
import { CookieOptions } from 'express';

const cookieOptions: CookieOptions = {
  httpOnly: true,
  secure: true,
  sameSite: 'strict', // or 'lax'
};

res.cookie('session', token, cookieOptions);
```

With `SameSite: 'strict'`, the browser never sends the cookie in cross-site requests, effectively eliminating most CSRF attack vectors.

### SameSite Values

| Value | Behavior | Use Case |
|-------|----------|----------|
| `strict` | Never sent cross-site | Maximum protection |
| `lax` | Sent with top-level navigation | Balance of security and usability |
| `none` | Always sent (requires Secure) | Cross-site scenarios |

### Token-Based Auth (No CSRF Needed)

If your API uses Bearer tokens in Authorization headers (not cookies), CSRF protection is unnecessary. The token must be explicitly added to requests, which attackers can't do from other sites.

```typescript
// This is NOT vulnerable to CSRF
headers: {
  'Authorization': 'Bearer ' + token
}
```

Since JavaScript on a different origin cannot read or set the Authorization header for your domain, Bearer token authentication is inherently immune to CSRF.

## Common Pitfalls

1. **Applying CSRF to stateless APIs**: Pure JWT APIs don't need CSRF. It's for cookie-based sessions.
2. **Forgetting SameSite**: Modern browsers support SameSite cookies, which provides significant CSRF protection.
3. **CSRF on GET requests**: CSRF protection is for state-changing methods (POST, PUT, DELETE). GETs should be idempotent.

## Best Practices

- Use SameSite=Strict or Lax for session cookies
- For traditional sessions, implement CSRF tokens
- For APIs with Bearer tokens, CSRF protection is unnecessary
- Always use HTTPS in production
- Validate Content-Type for JSON APIs

## Summary

- CSRF protection is essential for cookie-based authentication but unnecessary for Bearer-token APIs
- Use SameSite cookies (Strict or Lax) as the primary defense against cross-site request forgery
- Implement double-submit CSRF tokens with `csrf-csrf` for additional protection (replaces deprecated `csurf`)
- Only apply CSRF protection to state-changing methods (POST, PUT, DELETE)—GET should be idempotent

## Code Examples

**Implementing CSRF protection with csrf-csrf double-submit cookie pattern (replaces deprecated csurf)**

```typescript
import * as cookieParser from 'cookie-parser';
import { doubleCsrf } from 'csrf-csrf';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.use(cookieParser());
  const { doubleCsrfProtection } = doubleCsrf({
    getSecret: () => process.env.CSRF_SECRET,
    cookieName: '__csrf',
    cookieOptions: { sameSite: 'strict', secure: true },
  });
  app.use(doubleCsrfProtection);
  
  await app.listen(3000);
}
```


## Resources

- [Security Overview](https://docs.nestjs.com/security/csrf) — Security best practices

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*