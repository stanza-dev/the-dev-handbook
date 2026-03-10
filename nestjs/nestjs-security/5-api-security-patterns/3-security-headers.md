---
source_course: "nestjs-security"
source_lesson: "nestjs-security-security-headers"
---

# Security Headers Deep Dive

## Introduction

HTTP security headers instruct browsers on how to behave when handling your site's content. They're a critical defense layer against XSS, clickjacking, MIME sniffing, and other browser-based attacks.

## Key Concepts

- **Content-Security-Policy (CSP)**: Controls resource loading
- **Strict-Transport-Security (HSTS)**: Forces HTTPS
- **X-Frame-Options**: Prevents clickjacking
- **Permissions-Policy**: Controls browser features

## Real World Context

Security headers protect against:
- XSS via inline script injection (CSP)
- SSL stripping attacks (HSTS)
- Clickjacking via iframe embedding (X-Frame-Options)
- MIME type confusion (X-Content-Type-Options)

## Deep Dive

### Comprehensive Helmet Configuration

This production-ready configuration sets all major security headers with explicit directives tailored to a typical web application.

```typescript
import helmet from 'helmet';

app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'"], // Adjust based on needs
        styleSrc: ["'self'", "'unsafe-inline'", 'https://fonts.googleapis.com'],
        fontSrc: ["'self'", 'https://fonts.gstatic.com'],
        imgSrc: ["'self'", 'data:', 'https:'],
        connectSrc: ["'self'", 'https://api.example.com'],
        frameSrc: ["'none'"],
        objectSrc: ["'none'"],
        upgradeInsecureRequests: [],
      },
    },
    hsts: {
      maxAge: 31536000, // 1 year
      includeSubDomains: true,
      preload: true,
    },
    frameguard: { action: 'deny' },
    noSniff: true,
    xssFilter: true,
    referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  }),
);
```

The HSTS `preload: true` option allows your domain to be included in browser preload lists, ensuring HTTPS is enforced even on the first visit.

### Content-Security-Policy Explained

| Directive | Purpose | Example |
|-----------|---------|--------|
| `default-src` | Fallback for all fetch directives | `'self'` |
| `script-src` | Valid sources for JavaScript | `'self' 'unsafe-inline'` |
| `style-src` | Valid sources for CSS | `'self' fonts.googleapis.com` |
| `img-src` | Valid sources for images | `'self' data: https:` |
| `connect-src` | Valid targets for fetch/XHR | `'self' api.example.com` |
| `frame-src` | Valid sources for iframes | `'none'` |
| `report-uri` | URL to send violation reports | `/csp-report` |

### CSP Report-Only Mode

Deploy CSP in report-only mode first to discover which resources would be blocked without actually breaking functionality.

```typescript
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        // ... other directives
        reportUri: '/api/csp-report',
      },
      reportOnly: true, // Don't block, just report
    },
  }),
);

// Report endpoint
@Controller('api')
export class CspController {
  @Post('csp-report')
  handleCspReport(@Body() report: any) {
    this.logger.warn('CSP Violation:', report);
  }
}
```

Monitor the CSP reports for a few days to identify legitimate resources that need to be whitelisted before switching to enforcement mode.

### Permissions-Policy (Feature-Policy)

Disable browser features your application does not use to reduce the attack surface. This prevents embedded scripts from accessing these APIs.

```typescript
app.use((req, res, next) => {
  res.setHeader(
    'Permissions-Policy',
    'camera=(), microphone=(), geolocation=(), payment=()',
  );
  next();
});
```

The empty parentheses `()` disable the feature entirely. If you need camera access for specific origins, use `camera=(self "https://video.example.com")`.

### Custom Security Headers

Use an interceptor to remove identifying headers and disable caching on sensitive endpoints.

```typescript
@Injectable()
export class SecurityHeadersInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const response = context.switchToHttp().getResponse();
    
    // Remove server identification
    response.removeHeader('X-Powered-By');
    
    // Cache control for sensitive endpoints
    response.setHeader('Cache-Control', 'no-store, max-age=0');
    response.setHeader('Pragma', 'no-cache');
    
    // Prevent caching of sensitive data
    response.setHeader('Expires', '0');
    
    return next.handle();
  }
}
```

Removing `X-Powered-By` hides your tech stack from attackers. The `no-store` cache directive prevents browsers and proxies from caching sensitive responses.

### Environment-Specific Configuration

Disable restrictive headers in development to avoid frustrating debugging sessions. Apply full strictness only in production.

```typescript
const isProduction = process.env.NODE_ENV === 'production';

app.use(
  helmet({
    contentSecurityPolicy: isProduction ? {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'"], // Strict in production
      },
    } : false, // Disable in development for easier debugging
    hsts: isProduction, // Only in production (requires HTTPS)
  }),
);
```

HSTS must be disabled in development since local servers typically use HTTP. Enabling it on localhost would break your dev environment by forcing non-existent HTTPS.

## Common Pitfalls

1. **CSP breaking functionality**: Start with report-only mode to identify issues.
2. **HSTS on non-HTTPS**: HSTS only makes sense with HTTPS.
3. **Too restrictive CSP**: May break third-party integrations.

## Best Practices

- Start with Helmet defaults, customize as needed
- Use CSP report-only mode during development
- Test headers with securityheaders.com
- Document CSP exceptions with justifications
- Remove identifying headers (X-Powered-By)

## Summary

- Security headers instruct browsers on safe content handling—CSP, HSTS, X-Frame-Options, and more
- Use Helmet for sensible defaults and customize CSP directives for your specific resource needs
- Enable HSTS in production to force HTTPS and use report-only mode during development
- Remove identifying headers like `X-Powered-By` and set `Cache-Control: no-store` for sensitive endpoints

## Code Examples

**Configuring Content-Security-Policy and other security headers**

```typescript
import helmet from 'helmet';

app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'"], // Adjust based on needs
        styleSrc: ["'self'", "'unsafe-inline'", 'https://fonts.googleapis.com'],
        fontSrc: ["'self'", 'https://fonts.gstatic.com'],
        imgSrc: ["'self'", 'data:', 'https:'],
        connectSrc: ["'self'", 'https://api.example.com'],
        frameSrc: ["'none'"],
        objectSrc: ["'none'"],
        upgradeInsecureRequests: [],
      },
    },
    hsts: {
      maxAge: 31536000, // 1 year
      includeSubDomains: true,
      preload: true,
    },
    frameguard: { action: 'deny' },
    noSniff: true,
    xssFilter: true,
    referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  }),
);
```


## Resources

- [Helmet](https://docs.nestjs.com/security/helmet) — Security headers with Helmet

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*