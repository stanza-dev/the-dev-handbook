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

Enable CORS with a single method call. Without arguments, it allows requests from any origin — fine for development but not for production.

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors();
  await app.listen(3000);
}
```

This default configuration is permissive. For production, always pass an explicit configuration object.

### Production Configuration

Specify an explicit list of allowed origins, HTTP methods, and headers. Setting `credentials: true` enables cookies and Authorization headers in cross-origin requests.

```typescript
app.enableCors({
  origin: ['https://app.example.com', 'https://admin.example.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 3600,
});
```

The `maxAge: 3600` tells browsers to cache the preflight response for 1 hour, reducing the number of OPTIONS requests.

### Dynamic Origin Validation

For more flexibility, pass a function that programmatically validates origins — useful when allowed origins are loaded from a database or environment variables.

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

The `!origin` check allows server-to-server requests (which have no Origin header) to proceed. Remove it if you want to reject those too.

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

- CORS controls which domains can make cross-origin requests to your API
- Use `app.enableCors()` with an explicit list of allowed origins for production
- Never use `origin: '*'` with `credentials: true`—browsers reject this combination
- Use environment variables for allowed origins and test with actual browser requests, not just curl

## Code Examples

**Enabling CORS with default configuration using the built-in enableCors method**

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors();
  await app.listen(3000);
}
```


## Resources

- [CORS Documentation](https://docs.nestjs.com/security/cors) — Official CORS configuration guide

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*