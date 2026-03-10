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

NestJS automatically JSON-encodes responses, which handles most cases. The script tag below is treated as a harmless string in JSON.

```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  const user = { name: '<script>alert(1)</script>' };
  return user; // Safely encoded as JSON
}
// Output: {"name":"<script>alert(1)</script>"}
```

JSON encoding escapes the angle brackets, so even if malicious content is stored, it won't execute when parsed as JSON by the client.

### Explicit Encoding Utilities

For server-rendered HTML or non-JSON responses, use context-specific encoding functions to neutralize dangerous characters.

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

Each method targets a different output context — using HTML encoding in a JavaScript context (or vice versa) will not provide adequate protection.

### Response Transformation

Automate output encoding by applying `@Transform()` decorators on response DTOs. This ensures encoding happens consistently without manual effort in every controller.

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

The `@Expose()` decorator ensures only explicitly marked fields are included in the response, preventing accidental data leaks.

### Content Security Policy Headers

CSP headers add a browser-level defense layer. Even if encoding is missed, the browser blocks unauthorized script execution based on these directives.

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

Setting `objectSrc: ["'none'"]` blocks Flash and Java applets, while `upgradeInsecureRequests` automatically rewrites HTTP resource URLs to HTTPS.

### API Response Standards

Use an interceptor to enforce consistent response headers, including explicit content type and MIME sniffing protection.

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

The `nosniff` header prevents browsers from guessing the content type, which could lead to executing a JSON response as HTML.

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

- Output encoding provides defense in depth by escaping dangerous characters at display time
- NestJS JSON serialization automatically handles encoding for most API response cases
- Use CSP headers to prevent inline script execution and control resource loading in browsers
- For server-rendered HTML, apply context-aware encoding (HTML, JavaScript, URL) to user content

## Code Examples

**Encoding output data to prevent injection attacks in different contexts**

```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  const user = { name: '<script>alert(1)</script>' };
  return user; // Safely encoded as JSON
}
// Output: {"name":"<script>alert(1)</script>"}
```


## Resources

- [Helmet](https://docs.nestjs.com/security/helmet) — Security headers configuration

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*