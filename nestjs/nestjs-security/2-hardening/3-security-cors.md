---
source_course: "nestjs-security"
source_lesson: "nestjs-security-cors"
---

# CORS Configuration

## Introduction

Cross-Origin Resource Sharing (CORS) controls which domains can access your API. Without proper CORS configuration, browsers block requests from different origins—or worse, a misconfigured CORS policy can expose your API to unauthorized access.

## Key Concepts

- **CORS**: Browser security feature controlling cross-origin requests
- **Origin**: Protocol + domain + port (e.g., https://example.com:443)
- **Preflight Request**: OPTIONS request browsers send before actual request
- **Access-Control-Allow-Origin**: Header specifying allowed origins

## Real World Context

Your API runs at api.example.com, your frontend at app.example.com. Without CORS configuration, the browser blocks the frontend from calling the API. CORS tells the browser which origins are allowed.

## Deep Dive

### Basic CORS Setup

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors();
  await app.listen(3000);
}
```

### Production Configuration

```typescript
app.enableCors({
  origin: ['https://app.example.com', 'https://admin.example.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 3600,
});
```

### Dynamic Origin Validation

```typescript
app.enableCors({
  origin: (origin, callback) => {
    const allowedOrigins = [
      'https://app.example.com',
      'https://staging.example.com',
    ];
    
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
});
```

### CORS Options

| Option | Description | Example |
|--------|-------------|---------|
| `origin` | Allowed origins | `['https://example.com']` |
| `methods` | Allowed HTTP methods | `['GET', 'POST']` |
| `allowedHeaders` | Allowed request headers | `['Authorization']` |
| `exposedHeaders` | Headers exposed to browser | `['X-Total-Count']` |
| `credentials` | Allow cookies/auth headers | `true` |
| `maxAge` | Preflight cache duration (seconds) | `3600` |

## Common Pitfalls

1. **Using `origin: '*'` with credentials**: Browsers reject this combination. Be explicit about origins.
2. **Forgetting preflight for custom headers**: Requests with custom headers trigger preflight. Ensure OPTIONS is allowed.
3. **Environment-specific origins**: Don't hardcode. Use environment variables.

## Best Practices

- Never use `origin: '*'` in production
- Use environment variables for allowed origins
- Keep maxAge reasonable (1-24 hours)
- Only expose headers the frontend actually needs
- Test CORS with actual browser requests, not just curl

## Summary

CORS configuration controls which origins can access your API. Use `app.enableCors()` with explicit origins for production. Never use wildcards with credentials, and test thoroughly with actual browser requests.

## Resources

- [CORS Documentation](https://docs.nestjs.com/security/cors) — Official CORS configuration guide

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*