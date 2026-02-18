---
source_course: "nestjs-security"
source_lesson: "nestjs-security-output-encoding"
---

# Output Encoding

## Introduction

Even with input validation, output encoding provides defense in depth. When displaying user-generated content, encode it appropriately for the context (HTML, JavaScript, URL, CSS) to prevent XSS attacks.

## Key Concepts

- **Context-Aware Encoding**: Different contexts need different encoding
- **HTML Entity Encoding**: Convert `<` to `&lt;` for HTML context
- **JSON Encoding**: Proper escaping for JSON responses
- **URL Encoding**: Encode special characters in URLs

## Real World Context

Output encoding prevents:
- Stored XSS when displaying user content
- Reflected XSS in search results
- DOM-based XSS in single-page applications

## Deep Dive

### JSON Response Encoding

NestJS automatically JSON-encodes responses, which handles most cases:

```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  const user = { name: '<script>alert(1)</script>' };
  return user; // Safely encoded as JSON
}
// Output: {"name":"<script>alert(1)</script>"}
```

### Explicit Encoding Utilities

```typescript
import { escape } from 'html-escaper';

@Injectable()
export class EncodingService {
  htmlEncode(str: string): string {
    return escape(str);
    // < becomes &lt;
    // > becomes &gt;
    // & becomes &amp;
    // " becomes &quot;
    // ' becomes &#39;
  }

  urlEncode(str: string): string {
    return encodeURIComponent(str);
  }

  jsEncode(str: string): string {
    return JSON.stringify(str).slice(1, -1);
  }
}
```

### Response Transformation

```typescript
import { Expose, Transform } from 'class-transformer';
import { escape } from 'html-escaper';

export class UserResponseDto {
  @Expose()
  id: string;

  @Expose()
  @Transform(({ value }) => escape(value))
  name: string;

  @Expose()
  @Transform(({ value }) => escape(value))
  bio: string;
}

@Get(':id')
findOne(@Param('id') id: string) {
  const user = this.usersService.findOne(id);
  return new UserResponseDto(user);
}
```

### Content Security Policy Headers

```typescript
import helmet from 'helmet';

app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", 'data:', 'https:'],
        connectSrc: ["'self'"],
        fontSrc: ["'self'"],
        objectSrc: ["'none'"],
        upgradeInsecureRequests: [],
      },
    },
  }),
);
```

### API Response Standards

```typescript
@Injectable()
export class ResponseInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const response = context.switchToHttp().getResponse();
    
    // Always set JSON content type
    response.setHeader('Content-Type', 'application/json; charset=utf-8');
    
    // Prevent MIME sniffing
    response.setHeader('X-Content-Type-Options', 'nosniff');
    
    return next.handle();
  }
}
```

## Common Pitfalls

1. **Double encoding**: Encoding twice corrupts data. Encode once at output.
2. **Wrong context**: HTML encoding doesn't help in JavaScript contexts.
3. **Trusting frameworks blindly**: Know what your framework encodes and what it doesn't.

## Best Practices

- Let NestJS handle JSON encoding for API responses
- Use CSP headers to prevent inline script execution
- Encode user content before HTML rendering (server-side rendered apps)
- Always set proper Content-Type headers
- Use X-Content-Type-Options to prevent MIME sniffing

## Summary

Output encoding provides defense in depth. NestJS JSON serialization handles most API cases. Use CSP headers for browser security. For server-rendered HTML, encode user content appropriately for the context.

## Resources

- [Helmet](https://docs.nestjs.com/security/helmet) — Security headers configuration

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*