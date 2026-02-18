---
source_course: "nestjs-security"
source_lesson: "nestjs-security-helmet"
---

# Helmet

## Introduction

HTTP headers can make or break your application's security. Helmet is a collection of middleware that sets security-related HTTP headers to protect against well-known web vulnerabilities like clickjacking, XSS, and MIME sniffing.

## Key Concepts

- **Helmet**: Middleware that sets security HTTP headers
- **Content-Security-Policy (CSP)**: Controls which resources the browser can load
- **X-Frame-Options**: Prevents clickjacking by controlling iframe embedding
- **X-Content-Type-Options**: Prevents MIME type sniffing

## Real World Context

Without proper headers, your application is vulnerable to:
- **Clickjacking**: Attackers embed your site in an iframe and trick users
- **XSS**: Malicious scripts executing in users' browsers
- **MIME sniffing**: Browsers misinterpreting file types as executable

## Deep Dive

### Installation & Basic Setup

```bash
npm install helmet
```

```typescript
import helmet from 'helmet';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(helmet());
  await app.listen(3000);
}
```

### Headers Set by Helmet

| Header | Purpose | Default Value |
|--------|---------|---------------|
| `Content-Security-Policy` | Controls resource loading | Restrictive policy |
| `X-Frame-Options` | Prevents clickjacking | SAMEORIGIN |
| `X-Content-Type-Options` | Prevents MIME sniffing | nosniff |
| `X-DNS-Prefetch-Control` | Controls DNS prefetching | off |
| `Strict-Transport-Security` | Forces HTTPS | max-age=15552000 |
| `X-Download-Options` | IE download protection | noopen |
| `X-Permitted-Cross-Domain-Policies` | Adobe cross-domain | none |

### Custom Configuration

```typescript
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", 'data:', 'https:'],
        scriptSrc: ["'self'"],
      },
    },
    crossOriginEmbedderPolicy: false, // Disable if causing issues
  }),
);
```

### Disabling Specific Headers

```typescript
app.use(
  helmet({
    contentSecurityPolicy: false, // Disable CSP if managing separately
    crossOriginResourcePolicy: false,
  }),
);
```

## Common Pitfalls

1. **CSP breaking functionality**: Overly strict CSP blocks legitimate scripts. Test thoroughly and add necessary sources.
2. **Not configuring for your needs**: Default Helmet is restrictive. Configure for your specific requirements.
3. **Assuming Helmet is enough**: Helmet protects against header-based attacks, not application logic vulnerabilities.

## Best Practices

- Start with default Helmet and adjust as needed
- Use CSP report-only mode during development
- Test headers with tools like securityheaders.com
- Document any disabled protections with justifications
- Review headers after every deployment

## Summary

Helmet sets security HTTP headers with sensible defaults. Install it, use it with `app.use(helmet())`, and customize as needed. It's a simple but essential layer of defense against common web vulnerabilities.

## Resources

- [Helmet Documentation](https://docs.nestjs.com/security/helmet) — Official Helmet integration guide

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*