---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-middleware"
---

# Middleware

## Introduction

Middleware sits between incoming requests and your route handlers. It's the perfect place for cross-cutting concerns like logging, authentication checks, request transformation, and rate limiting.

## Key Concepts

- **Middleware**: Functions executed before route handlers
- **NestMiddleware**: Interface for class-based middleware
- **next()**: Function that passes control to the next middleware or handler
- **MiddlewareConsumer**: Service for configuring middleware in modules

## Real World Context

Production APIs use middleware for:
- **Request logging**: Log every request with timing and user agent
- **Authentication checks**: Verify tokens before reaching protected routes
- **Request ID injection**: Add unique IDs for distributed tracing

## Deep Dive

### Request Lifecycle Order

```
Request → Middleware → Guards → Interceptors (pre) → Pipes → Handler → Interceptors (post) → Response
```

### Class-Based Middleware

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const start = Date.now();
    res.on('finish', () => {
      console.log(`${req.method} ${req.url} - ${Date.now() - start}ms`);
    });
    next();
  }
}
```

### Applying Middleware

```typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';

@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('*');
  }
}
```

### Route-Specific Middleware

```typescript
consumer
  .apply(AuthMiddleware)
  .exclude({ path: 'health', method: RequestMethod.GET })
  .forRoutes({ path: 'users', method: RequestMethod.ALL });
```

### Functional Middleware

For simple cases without dependencies:

```typescript
export function logger(req: Request, res: Response, next: NextFunction) {
  console.log('Request...');
  next();
}
```

## Common Pitfalls

1. **Forgetting to call `next()`**: The request hangs forever without calling `next()`. Always call it unless you're ending the response.
2. **Order matters**: Middleware executes in the order applied. Auth should come before logging if you want user info in logs.
3. **Blocking the event loop**: Synchronous heavy operations block all requests. Use async operations.

## Best Practices

- Use functional middleware for simple cases without DI needs
- Inject services when needed via class middleware
- Handle errors properly—throw HttpExceptions for the exception filter
- Keep middleware focused on one responsibility

## Summary

Middleware executes before route handlers, perfect for logging, auth checks, and request transformation. Implement `NestMiddleware` for class-based middleware, or use simple functions. Configure in the module's `configure()` method using `MiddlewareConsumer`.

## Resources

- [Middleware Documentation](https://docs.nestjs.com/middleware) — Official guide to implementing middleware

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*