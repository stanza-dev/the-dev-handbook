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

### Cookie-Based CSRF Protection

```bash
npm install csurf cookie-parser
```

```typescript
import * as cookieParser from 'cookie-parser';
import * as csurf from 'csurf';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.use(cookieParser());
  app.use(csurf({ cookie: true }));
  
  await app.listen(3000);
}
```

### Getting CSRF Token

```typescript
@Controller()
export class AppController {
  @Get('csrf-token')
  getCsrfToken(@Req() req): { csrfToken: string } {
    return { csrfToken: req.csrfToken() };
  }
}
```

### SameSite Cookies (Modern Approach)

```typescript
import { CookieOptions } from 'express';

const cookieOptions: CookieOptions = {
  httpOnly: true,
  secure: true,
  sameSite: 'strict', // or 'lax'
};

res.cookie('session', token, cookieOptions);
```

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

CSRF protection is essential for cookie-based authentication. Use SameSite cookies as the first line of defense, and implement CSRF tokens for additional security. Token-based APIs (Bearer tokens) don't need CSRF protection since tokens can't be automatically sent cross-site.

## Resources

- [Security Overview](https://docs.nestjs.com/security/csrf) — Security best practices

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*