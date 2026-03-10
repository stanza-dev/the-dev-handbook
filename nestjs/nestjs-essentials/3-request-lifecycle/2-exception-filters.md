---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-exception-filters"
---

# Exception Filters in the Request Pipeline

## Introduction

When an error occurs during request processing, NestJS's exception layer catches it and produces a meaningful HTTP response. Understanding where exception filters sit in the request pipeline is key to building robust error handling.

## Key Concepts

- **Exception Layer**: NestJS's built-in mechanism for catching unhandled exceptions
- **Default Exception Filter**: Catches all `HttpException` instances and formats the response
- **Pipeline Position**: Exception filters execute when any component (guard, pipe, interceptor, handler) throws

## Real World Context

In a production API, exceptions can be thrown from anywhere in the pipeline — a guard denying access, a pipe rejecting invalid input, or a service encountering a missing resource. The exception filter layer catches all of these and ensures a consistent JSON error response reaches the client.

## Deep Dive

### Where Filters Sit in the Pipeline

```
Request → Middleware → Guards → Interceptors → Pipes → Handler
                         ↓         ↓          ↓        ↓
                    Exception Filter catches errors from ANY stage
```

If an exception is thrown at any point, the rest of the pipeline is skipped and the exception filter handles it.

### Built-in Exception Handling

NestJS provides a default exception filter that handles `HttpException` and its subclasses:

```typescript
// These are automatically caught and formatted:
throw new BadRequestException('Invalid email format');
// → { statusCode: 400, message: 'Invalid email format', error: 'Bad Request' }

throw new NotFoundException('User not found');
// → { statusCode: 404, message: 'User not found', error: 'Not Found' }
```

### Complete List of Built-in Exceptions

| Exception | Status Code | Use Case |
|-----------|-------------|----------|
| `BadRequestException` | 400 | Invalid input |
| `UnauthorizedException` | 401 | Missing/invalid credentials |
| `ForbiddenException` | 403 | Insufficient permissions |
| `NotFoundException` | 404 | Resource not found |
| `MethodNotAllowedException` | 405 | Wrong HTTP method |
| `ConflictException` | 409 | Resource conflict |
| `PayloadTooLargeException` | 413 | Request body too large |
| `UnprocessableEntityException` | 422 | Semantic validation failure |
| `InternalServerErrorException` | 500 | Unexpected server error |

### Unhandled Exceptions

Exceptions that are **not** `HttpException` subclasses produce a generic 500 response:

```typescript
// This produces: { statusCode: 500, message: 'Internal server error' }
throw new Error('Database connection failed');
```

To customize this behavior, you'll create custom exception filters (covered in the Exception Handling section).

## Common Pitfalls

1. **Throwing plain Error objects**: They become generic 500 responses. Use `HttpException` subclasses for proper status codes.
2. **Exposing internal errors**: Stack traces and internal details leak in development mode. Configure production error handling to hide them.
3. **Catching too broadly**: `try/catch` blocks in handlers prevent exception filters from running. Let exceptions propagate to the filter layer.

## Best Practices

- Use built-in exception classes for standard HTTP errors
- Let exceptions propagate — don't swallow them in try/catch
- Reserve custom exception filters for advanced formatting (covered next section)

## Summary

Exception filters catch errors thrown from any part of the request pipeline. NestJS's default filter handles `HttpException` subclasses automatically. Use built-in exceptions like `NotFoundException` and `BadRequestException` for common errors. Custom exception filters allow advanced formatting and logging.

## Code Examples

**A custom exception filter — @Catch(HttpException) specifies which exceptions to handle, ArgumentsHost gives access to the request/response context**

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      message: exception.message,
      timestamp: new Date().toISOString(),
    });
  }
}
```


## Resources

- [Exception Filters Documentation](https://docs.nestjs.com/exception-filters) — Official guide to exception handling

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*