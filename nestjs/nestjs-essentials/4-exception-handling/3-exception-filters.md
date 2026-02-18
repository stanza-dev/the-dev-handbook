---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-custom-exception-filters"
---

# Exception Filters

## Introduction

Exception filters catch exceptions and transform them into HTTP responses. They provide centralized error handling, logging, and consistent response formatting.

## Key Concepts

- **@Catch()**: Decorator specifying which exceptions to catch
- **ExceptionFilter**: Interface for filter implementation
- **ArgumentsHost**: Access to request/response context

## Deep Dive

### Basic Exception Filter

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: ctx.getRequest().url,
      message: exception.message,
    });
  }
}
```

### Catch All Exceptions

```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      message: exception instanceof HttpException ? exception.message : 'Internal server error',
    });
  }
}
```

### Applying Filters

```typescript
// Method level
@Post()
@UseFilters(HttpExceptionFilter)
create(@Body() dto: CreateUserDto) {}

// Controller level
@Controller('users')
@UseFilters(HttpExceptionFilter)
export class UsersController {}

// Global with DI
@Module({
  providers: [{ provide: APP_FILTER, useClass: AllExceptionsFilter }],
})
export class AppModule {}
```

## Common Pitfalls

1. **Forgetting @Catch()**: Without args, catches everything.
2. **Missing DI**: Use APP_FILTER provider for dependency injection.
3. **Circular logging**: Don't throw in exception filters.

## Best Practices

- Create a global catch-all filter for unhandled exceptions
- Log errors with full context
- Never expose internal details to clients

## Summary

Exception filters provide centralized error handling. Use @Catch() to specify types and apply globally for consistent responses.

## Resources

- [Exception Filters](https://docs.nestjs.com/exception-filters) — Official exception filters guide

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*