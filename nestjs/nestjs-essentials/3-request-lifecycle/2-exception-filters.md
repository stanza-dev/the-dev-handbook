---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-exception-filters"
---

# Exception Filters

## Introduction

Errors happen—invalid data, missing resources, permission denied. How your API handles errors determines the developer experience for consumers. NestJS provides a powerful exception layer that catches errors and transforms them into meaningful HTTP responses.

## Key Concepts

- **Exception Filter**: A class that catches exceptions and transforms them into responses
- **HttpException**: Base class for HTTP-related exceptions
- **@Catch()**: Decorator that specifies which exceptions a filter handles
- **ArgumentsHost**: Provides access to the request/response context

## Real World Context

APIs must return consistent error responses. Whether it's a 404 Not Found, 400 Bad Request, or 500 Internal Server Error, clients need predictable JSON structures to handle errors gracefully. Exception filters centralize this logic.

## Deep Dive

### Built-in Exceptions

NestJS provides common HTTP exceptions:

```typescript
throw new BadRequestException('Invalid email format');
throw new UnauthorizedException('Invalid token');
throw new ForbiddenException('Insufficient permissions');
throw new NotFoundException('User not found');
throw new ConflictException('Email already exists');
throw new InternalServerErrorException('Database connection failed');
```

### Custom Exception Response

```typescript
throw new HttpException({
  status: HttpStatus.FORBIDDEN,
  error: 'Custom error message',
  code: 'FORBIDDEN_ACCESS',
}, HttpStatus.FORBIDDEN);
```

### Creating a Custom Exception Filter

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message,
    });
  }
}
```

### Applying Filters

```typescript
// Method level
@Post()
@UseFilters(HttpExceptionFilter)
create() { ... }

// Controller level
@Controller('cats')
@UseFilters(HttpExceptionFilter)
export class CatsController {}

// Global level (main.ts)
app.useGlobalFilters(new HttpExceptionFilter());
```

### Catch-All Filter

```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    // Handle any exception type
  }
}
```

## Common Pitfalls

1. **Exposing internal errors**: Never expose stack traces or internal details in production. Log them server-side, return generic messages to clients.
2. **Inconsistent error formats**: Different endpoints returning different error shapes confuses API consumers. Use global filters for consistency.
3. **Swallowing exceptions**: Catch-all filters that don't log lose valuable debugging information.

## Best Practices

- Use global exception filters for consistent error responses
- Log exceptions with context (request ID, user ID, path)
- Return machine-readable error codes alongside messages
- Never expose sensitive information in error responses

## Summary

Exception filters catch errors and transform them into HTTP responses. Use built-in exceptions like `NotFoundException` for common cases, or create custom filters for advanced formatting. Apply filters globally for consistent error handling across your API.

## Resources

- [Exception Filters Documentation](https://docs.nestjs.com/exception-filters) — Official guide to exception handling

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*