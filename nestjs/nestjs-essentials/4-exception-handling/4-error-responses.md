---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-error-responses"
---

# Error Response Patterns

## Introduction

Consistent error responses improve API usability. A well-designed error format helps clients understand what went wrong and how to fix it.

## Key Concepts

- **Error Schema**: Consistent structure for all errors
- **Error Codes**: Machine-readable identifiers
- **Validation Errors**: Field-level error details

## Deep Dive

### Standard Error Response

```typescript
interface ErrorResponse {
  statusCode: number;
  message: string;
  error: string;
  timestamp: string;
  path: string;
  errorCode?: string;
}
```

### Validation Error Response

```typescript
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    { "field": "email", "message": "must be a valid email" },
    { "field": "password", "message": "must be at least 8 characters" }
  ]
}
```

### Custom Validation Filter

```typescript
@Catch(BadRequestException)
export class ValidationExceptionFilter implements ExceptionFilter {
  catch(exception: BadRequestException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const exceptionResponse = exception.getResponse() as any;

    const errors = Array.isArray(exceptionResponse.message)
      ? exceptionResponse.message.map((msg: string) => {
          const [field, ...rest] = msg.split(' ');
          return { field, message: rest.join(' ') };
        })
      : [{ field: 'unknown', message: exceptionResponse.message }];

    response.status(400).json({ statusCode: 400, message: 'Validation failed', errors });
  }
}
```

### Error Code Enum

```typescript
export const ErrorCodes = {
  INVALID_CREDENTIALS: 'AUTH001',
  TOKEN_EXPIRED: 'AUTH002',
  USER_NOT_FOUND: 'USER001',
  EMAIL_ALREADY_EXISTS: 'USER002',
} as const;
```

## Common Pitfalls

1. **Inconsistent formats**: Use one format across all endpoints.
2. **Missing error codes**: Codes help programmatic handling.
3. **Vague messages**: Be specific about what went wrong.

## Best Practices

- Define a consistent error schema and document it
- Use error codes for programmatic handling
- Provide field-level details for validation errors

## Summary

Consistent error responses improve API usability. Define a standard schema, include error codes, and provide detailed validation errors.

## Resources

- [Exception Filters](https://docs.nestjs.com/exception-filters) — Error handling patterns

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*