---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-param-decorators"
---

# Custom Parameter Decorators

## Introduction

Parameter decorators extract data from the request context and inject it into your handler methods. Instead of accessing `request.user` everywhere, create a `@CurrentUser()` decorator for clean, testable code.

## Key Concepts

- **createParamDecorator()**: NestJS factory for parameter decorators
- **ExecutionContext**: Access to request, response, and handler metadata
- **Data parameter**: Optional argument passed to the decorator
- **Type safety**: Return properly typed values from decorators

## Real World Context

Parameter decorators eliminate repetitive request access patterns:
- `@CurrentUser()` instead of `request.user`
- `@ClientIP()` instead of parsing headers
- `@Pagination()` instead of manually extracting page/limit
- `@TenantId()` for multi-tenant applications

## Deep Dive

### Basic Parameter Decorator

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);

// Usage
@Get('profile')
getProfile(@CurrentUser() user: User) {
  return user;
}
```

### With Data Parameter

```typescript
export const CurrentUser = createParamDecorator(
  (data: keyof User | undefined, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    return data ? user?.[data] : user;
  },
);

// Usage
@Get('profile')
getProfile(@CurrentUser() user: User) { ... }

@Get('email')
getEmail(@CurrentUser('email') email: string) { ... }
```

### Extracting Client IP

```typescript
export const ClientIP = createParamDecorator(
  (data: unknown, ctx: ExecutionContext): string => {
    const request = ctx.switchToHttp().getRequest();
    const forwarded = request.headers['x-forwarded-for'];
    return forwarded
      ? forwarded.split(',')[0].trim()
      : request.ip;
  },
);
```

### Pagination Decorator

```typescript
export type PaginationParams = {
  page: number;
  limit: number;
  skip: number;
};

export const Pagination = createParamDecorator(
  (data: { maxLimit?: number } = {}, ctx: ExecutionContext): PaginationParams => {
    const request = ctx.switchToHttp().getRequest();
    const maxLimit = data.maxLimit ?? 100;
    
    const page = Math.max(1, parseInt(request.query.page) || 1);
    const limit = Math.min(maxLimit, Math.max(1, parseInt(request.query.limit) || 10));
    const skip = (page - 1) * limit;
    
    return { page, limit, skip };
  },
);

// Usage
@Get()
findAll(@Pagination({ maxLimit: 50 }) pagination: PaginationParams) {
  return this.service.findAll(pagination);
}
```

### Combining with Pipes

```typescript
import { ValidationPipe } from '@nestjs/common';

@Get()
findOne(
  @CurrentUser(new ValidationPipe({ validateCustomDecorators: true }))
  user: ValidatedUserDto,
) { ... }
```

## Common Pitfalls

1. **Assuming user exists**: Always handle the case where `request.user` is undefined.
2. **Not handling all transports**: If your app uses WebSockets or microservices, use `ctx.getType()` to handle different contexts.
3. **Heavy operations**: Decorators run on every request. Keep them fast.

## Best Practices

- Return `undefined` or throw for missing data, don't return empty objects
- Add TypeScript types for decorator return values
- Use the data parameter for flexible extraction
- Consider caching expensive lookups

## Summary

Create parameter decorators with `createParamDecorator()`. Access the request via `ExecutionContext`. Use the data parameter for flexible extraction like `@CurrentUser('email')`. Keep decorators fast and handle edge cases.

## Resources

- [Custom Route Decorators](https://docs.nestjs.com/custom-decorators#custom-route-decorators) — Creating custom parameter decorators

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*