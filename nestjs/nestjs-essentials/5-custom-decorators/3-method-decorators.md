---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-method-decorators"
---

# Custom Method & Class Decorators

## Introduction

Method and class decorators attach metadata or modify behavior at the handler or controller level. They're perfect for combining multiple decorators, adding Swagger documentation, or creating domain-specific annotations.

## Key Concepts

- **Method Decorator**: Modifies or annotates a single route handler
- **Class Decorator**: Applies to an entire controller or provider
- **applyDecorators()**: NestJS utility to combine multiple decorators
- **Reflector**: Service to read decorator metadata

## Real World Context

Method/class decorators simplify common patterns:
- `@Auth('admin')` combines `@UseGuards()`, `@ApiBearerAuth()`, `@Roles()`
- `@Cached(60)` combines `@UseInterceptors(CacheInterceptor)` with `@CacheTTL(60)`
- `@Audit('USER_ACTION')` marks endpoints for audit logging

## Deep Dive

### Combining Decorators with applyDecorators

```typescript
import { applyDecorators, UseGuards, SetMetadata } from '@nestjs/common';
import { ApiBearerAuth, ApiUnauthorizedResponse } from '@nestjs/swagger';

export function Auth(...roles: string[]) {
  return applyDecorators(
    SetMetadata('roles', roles),
    UseGuards(AuthGuard, RolesGuard),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Unauthorized' }),
  );
}

// Usage - replaces 4 decorators with 1
@Auth('admin')
@Post()
createUser() { ... }
```

### Creating Metadata Decorators

```typescript
import { Reflector } from '@nestjs/core';

// Using Reflector.createDecorator (NestJS 10+)
export const Roles = Reflector.createDecorator<string[]>();

// In guard
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get(Roles, context.getHandler());
    if (!roles) return true;
    
    const request = context.switchToHttp().getRequest();
    return roles.some(role => request.user?.roles?.includes(role));
  }
}
```

### Class-Level Decorator

```typescript
import { Controller, applyDecorators } from '@nestjs/common';
import { ApiTags } from '@nestjs/swagger';

export function ApiController(prefix: string, tag?: string) {
  return applyDecorators(
    Controller(prefix),
    ApiTags(tag ?? prefix),
  );
}

// Usage
@ApiController('users', 'User Management')
export class UsersController { ... }
```

### Conditional Decorators

```typescript
export function ConditionalCache(condition: boolean, ttl: number) {
  if (condition) {
    return applyDecorators(
      UseInterceptors(CacheInterceptor),
      CacheTTL(ttl),
    );
  }
  return () => {}; // No-op decorator
}

// Usage
@ConditionalCache(process.env.ENABLE_CACHE === 'true', 60)
@Get()
findAll() { ... }
```

### Reading Metadata from Multiple Sources

```typescript
@Injectable()
export class MetadataGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Check method first, then class
    const roles = this.reflector.getAllAndOverride<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);
    
    // Or merge from both
    const allRoles = this.reflector.getAllAndMerge<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);
    
    return this.validateRoles(roles);
  }
}
```

## Common Pitfalls

1. **Decorator order in applyDecorators**: Order matters. Guards should come before Swagger decorators.
2. **Forgetting to read from class**: Method-level metadata doesn't include class-level. Use `getAllAndOverride`.
3. **Heavy decorator factories**: Factory functions run at startup. Keep them synchronous and fast.

## Best Practices

- Use `applyDecorators()` to create semantic, domain-specific decorators
- Use `Reflector.createDecorator()` for type-safe metadata
- Document composite decorators so team knows what they include
- Test decorators by testing guards/interceptors that read their metadata

## Summary

Method and class decorators annotate handlers and controllers. Use `applyDecorators()` to combine multiple decorators into one semantic decorator. Read metadata with `Reflector` in guards and interceptors. Use `getAllAndOverride()` to check both method and class levels.

## Resources

- [Custom Decorators](https://docs.nestjs.com/custom-decorators) — Full guide to custom decorators

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*