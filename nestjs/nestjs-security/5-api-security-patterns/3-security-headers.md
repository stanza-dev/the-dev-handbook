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

### Permissions-Policy (Feature-Policy)

```typescript
app.use((req, res, next) => {
  res.setHeader(
    'Permissions-Policy',
    'camera=(), microphone=(), geolocation=(), payment=()',
  );
  next();
});
```

### Custom Security Headers

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

### Environment-Specific Configuration

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

Security headers provide browser-level protection. Use Helmet for sensible defaults, customize CSP for your resource needs, and enable HSTS in production. Test with report-only mode before enforcing strict policies.

## Resources

- [Helmet](https://docs.nestjs.com/security/helmet) — Security headers with Helmet

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*