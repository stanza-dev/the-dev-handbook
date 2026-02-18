---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-custom-exceptions"
---

# Custom Exceptions

## Introduction

Built-in exceptions cover common cases, but your domain may need specific error types. Custom exceptions extend HttpException for domain-specific error handling.

## Key Concepts

- **Custom Exception Class**: Extends HttpException
- **Domain Errors**: Business logic specific exceptions
- **Error Codes**: Machine-readable error identifiers

## Deep Dive

### Basic Custom Exception

```typescript
import { HttpException, HttpStatus } from '@nestjs/common';

export class UserNotFoundException extends HttpException {
  constructor(userId: string) {
    super(`User with ID ${userId} not found`, HttpStatus.NOT_FOUND);
  }
}
```

### Exception with Error Code

```typescript
export class BusinessException extends HttpException {
  constructor(
    message: string,
    errorCode: string,
    statusCode: HttpStatus = HttpStatus.BAD_REQUEST,
  ) {
    super({ statusCode, message, errorCode, timestamp: new Date().toISOString() }, statusCode);
  }
}

export class InsufficientFundsException extends BusinessException {
  constructor(required: number, available: number) {
    super(`Insufficient funds: required ${required}, available ${available}`, 'INSUFFICIENT_FUNDS', HttpStatus.PAYMENT_REQUIRED);
  }
}
```

### Exception Hierarchy

```typescript
export abstract class DomainException extends HttpException {
  abstract readonly errorCode: string;
}

export class OrderNotFoundException extends DomainException {
  errorCode = 'ORDER_NOT_FOUND';
  constructor(orderId: string) {
    super(`Order ${orderId} not found`, HttpStatus.NOT_FOUND);
  }
}
```

## Common Pitfalls

1. **Too many exceptions**: Don't create one for every possible error.
2. **Inconsistent format**: Keep error response structure consistent.
3. **Missing context**: Include relevant IDs in messages.

## Best Practices

- Create domain-specific exception hierarchies
- Include error codes for programmatic handling
- Add relevant context to messages

## Summary

Custom exceptions extend HttpException for domain-specific errors. Create hierarchies for related errors and include error codes for programmatic handling.

## Resources

- [Exception Filters](https://docs.nestjs.com/exception-filters) — Custom exceptions guide

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*