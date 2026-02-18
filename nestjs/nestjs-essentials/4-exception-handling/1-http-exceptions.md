---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-http-exceptions"
---

# HTTP Exceptions

## Introduction

APIs must communicate errors clearly. NestJS provides built-in HTTP exception classes that automatically format error responses with proper status codes.

## Key Concepts

- **HttpException**: Base class for all HTTP exceptions
- **Built-in Exceptions**: BadRequestException, NotFoundException, etc.
- **Status Codes**: HTTP status codes (4xx client errors, 5xx server errors)

## Deep Dive

### Basic HttpException

```typescript
import { HttpException, HttpStatus } from '@nestjs/common';

@Get(':id')
findOne(@Param('id') id: string) {
  const user = this.usersService.findOne(id);
  if (!user) {
    throw new HttpException('User not found', HttpStatus.NOT_FOUND);
  }
  return user;
}
```

### Built-in Exception Classes

```typescript
import {
  BadRequestException,
  UnauthorizedException,
  ForbiddenException,
  NotFoundException,
  ConflictException,
} from '@nestjs/common';

throw new BadRequestException('Invalid input');
throw new UnauthorizedException('Invalid credentials');
throw new ForbiddenException('Access denied');
throw new NotFoundException('Resource not found');
throw new ConflictException('Email already exists');
```

### Custom Error Messages

```typescript
throw new BadRequestException({
  message: 'Validation failed',
  errors: ['email must be valid', 'password too short'],
});
```

## Common Pitfalls

1. **Wrong status codes**: Use 404 for missing, 400 for bad input, 401 for auth.
2. **Exposing internal errors**: Don't leak stack traces to clients.
3. **Catching and re-throwing**: Don't catch just to re-throw unchanged.

## Best Practices

- Use specific exception classes over generic HttpException
- Include helpful error messages for debugging
- Never expose sensitive information in responses

## Summary

NestJS provides built-in HTTP exceptions for common error scenarios. Use specific classes like NotFoundException for clear, consistent error responses.

## Resources

- [Exception Filters](https://docs.nestjs.com/exception-filters) — Official exception handling guide

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*