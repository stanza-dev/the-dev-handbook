---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-domain-errors"
---

# Domain Error Hierarchy

## Introduction

Not all errors are created equal. HTTP errors, validation errors, and business rule violations need different handling. A domain error hierarchy makes error handling predictable, type-safe, and meaningful.

## Key Concepts

- **Domain Error**: Business logic violation (e.g., InsufficientFundsError)
- **Application Error**: Use case failures (e.g., UserNotFoundError)
- **Infrastructure Error**: External system failures (e.g., DatabaseConnectionError)
- **Error Mapping**: Converting domain errors to HTTP responses

## Real World Context

Structured errors enable:
- Consistent API error responses
- Type-safe error handling in services
- Clear separation between error types
- Easy logging and monitoring

## Deep Dive

### Base Error Classes

```typescript
export abstract class DomainError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;
  
  constructor(message: string) {
    super(message);
    this.name = this.constructor.name;
  }
}

export abstract class ValidationError extends DomainError {
  readonly statusCode = 400;
}

export abstract class NotFoundError extends DomainError {
  readonly statusCode = 404;
}

export abstract class ConflictError extends DomainError {
  readonly statusCode = 409;
}

export abstract class BusinessRuleError extends DomainError {
  readonly statusCode = 422;
}
```

### Specific Domain Errors

```typescript
export class UserNotFoundError extends NotFoundError {
  readonly code = 'USER_NOT_FOUND';
  
  constructor(userId: string) {
    super(`User with ID ${userId} not found`);
  }
}

export class EmailAlreadyExistsError extends ConflictError {
  readonly code = 'EMAIL_ALREADY_EXISTS';
  
  constructor(email: string) {
    super(`User with email ${email} already exists`);
  }
}

export class InsufficientFundsError extends BusinessRuleError {
  readonly code = 'INSUFFICIENT_FUNDS';
  
  constructor(
    public readonly required: number,
    public readonly available: number,
  ) {
    super(`Insufficient funds: required ${required}, available ${available}`);
  }
}

export class OrderCannotBeCancelledError extends BusinessRuleError {
  readonly code = 'ORDER_CANNOT_BE_CANCELLED';
  
  constructor(orderId: string, status: string) {
    super(`Order ${orderId} cannot be cancelled in status ${status}`);
  }
}
```

### Service Using Domain Errors

```typescript
@Injectable()
export class OrdersService {
  async cancelOrder(orderId: string): Promise<void> {
    const order = await this.ordersRepository.findById(orderId);
    
    if (!order) {
      throw new OrderNotFoundError(orderId);
    }
    
    if (!order.canBeCancelled()) {
      throw new OrderCannotBeCancelledError(orderId, order.status);
    }
    
    order.cancel();
    await this.ordersRepository.save(order);
  }
}
```

### Global Domain Error Filter

```typescript
@Catch(DomainError)
export class DomainErrorFilter implements ExceptionFilter {
  catch(exception: DomainError, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();

    response.status(exception.statusCode).json({
      statusCode: exception.statusCode,
      errorCode: exception.code,
      message: exception.message,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}
```

### Result Type Pattern (Alternative)

```typescript
export type Result<T, E = Error> =
  | { success: true; value: T }
  | { success: false; error: E };

export const ok = <T>(value: T): Result<T, never> => ({
  success: true,
  value,
});

export const fail = <E>(error: E): Result<never, E> => ({
  success: false,
  error,
});

// Usage
async findUser(id: string): Promise<Result<User, UserNotFoundError>> {
  const user = await this.repository.findById(id);
  return user ? ok(user) : fail(new UserNotFoundError(id));
}
```

## Common Pitfalls

1. **Generic errors everywhere**: "Something went wrong" isn't helpful. Be specific.
2. **HTTP errors in domain**: Domain shouldn't know about HTTP.
3. **Swallowing errors**: Always handle or re-throw.

## Best Practices

- Create specific error classes for each failure case
- Include error codes for programmatic handling
- Map domain errors to HTTP in filters
- Include relevant context in error messages
- Document error codes for API consumers

## Summary

Domain error hierarchies provide type-safe, meaningful error handling. Create base classes for error categories, specific classes for each failure case, and global filters for HTTP mapping. Include error codes and context for debugging.

## Resources

- [Exception Filters](https://docs.nestjs.com/exception-filters) — Error handling strategies

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*