---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-interceptors"
---

# Interceptors

## Introduction

Interceptors add extra logic before and after method execution. They can transform the result returned from a handler, transform exceptions, extend basic method behavior, or even completely override a method. Think of them as aspect-oriented programming for NestJS.

## Key Concepts

- **Interceptor**: A class that implements `NestInterceptor` to add logic around method execution
- **CallHandler**: Represents the route handler; calling `handle()` invokes it
- **RxJS Observable**: Interceptors work with Observables for powerful transformations
- **Execution Context**: Provides metadata about the current request

## Real World Context

Common interceptor use cases:
- **Logging**: Log request/response times and payloads
- **Caching**: Return cached responses for expensive operations
- **Response transformation**: Wrap all responses in a standard format
- **Timeout**: Cancel requests that take too long

## Deep Dive

### Basic Interceptor Structure

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap, map } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');
    const now = Date.now();
    
    return next
      .handle()
      .pipe(
        tap(() => console.log(`After... ${Date.now() - now}ms`)),
      );
  }
}
```

### Response Transformation

Wrap all responses in a standard format:

```typescript
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}
```

### Timeout Interceptor

```typescript
import { timeout, catchError } from 'rxjs/operators';
import { TimeoutError, throwError } from 'rxjs';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000),
      catchError(err => {
        if (err instanceof TimeoutError) {
          throw new RequestTimeoutException();
        }
        return throwError(() => err);
      }),
    );
  }
}
```

### Applying Interceptors

```typescript
// Method level
@UseInterceptors(LoggingInterceptor)
@Get()
findAll() { ... }

// Controller level
@Controller('cats')
@UseInterceptors(LoggingInterceptor)
export class CatsController {}

// Global level
app.useGlobalInterceptors(new LoggingInterceptor());
```

### Execution Order

When multiple interceptors are applied:
1. Global interceptors execute first (in registration order)
2. Controller interceptors execute next
3. Method interceptors execute last
4. Response flows back in reverse order

## Common Pitfalls

1. **Not returning the Observable**: You must return `next.handle()` or a derived Observable. Forgetting this breaks the request chain.
2. **Blocking operations**: Don't do synchronous heavy work—it blocks the event loop.
3. **Modifying request in 'after'**: The `tap()` operator runs after the response. To modify the request, do it before `next.handle()`.

## Best Practices

- Use `map()` to transform responses
- Use `tap()` for side effects like logging
- Use `catchError()` for exception transformation
- Keep interceptors focused on one concern

## Summary

Interceptors wrap route handler execution, enabling before/after logic, response transformation, caching, and timeouts. They work with RxJS Observables for powerful stream manipulation. Apply them at method, controller, or global level.

## Resources

- [Interceptors Documentation](https://docs.nestjs.com/interceptors) — Official guide to implementing interceptors

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*