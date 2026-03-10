---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-guards"
---

# Guards

## Introduction

Guards determine whether a request should be handled by the route handler. They are the authorization layer of NestJS — deciding if the current user has permission to access a specific endpoint. Unlike middleware, guards have access to the `ExecutionContext` and know exactly which handler will execute next.

## Key Concepts

- **Guard**: A class implementing `CanActivate` that returns true/false to allow/deny access
- **CanActivate**: Interface requiring a `canActivate(context)` method
- **ExecutionContext**: Extended `ArgumentsHost` with `getHandler()` and `getClass()` methods
- **@UseGuards()**: Decorator to apply guards at method, controller, or global level

## Real World Context

Production APIs need authorization:
- **Role-based access**: Only admins can delete users
- **Ownership checks**: Users can only edit their own profiles
- **Feature flags**: Premium features restricted to paying users
- **Rate limiting**: Throttle requests per user/IP

Guards centralize this logic instead of scattering `if (user.role !== 'admin')` checks across every handler.

## Deep Dive

### Guard vs Middleware

Middleware doesn't know which handler will execute — it just calls `next()`. Guards have full context:

```typescript
// Middleware: "dumb" — doesn't know the destination
use(req, res, next) { next(); }

// Guard: "smart" — knows the handler and can read its metadata
canActivate(context: ExecutionContext) {
  const handler = context.getHandler(); // The route method
  const controller = context.getClass(); // The controller class
}
```

### Basic Auth Guard

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization;
    return !!token; // Allow if token exists
  }
}
```

### Role-Based Guard with Metadata

```typescript
import { Reflector } from '@nestjs/core';
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!requiredRoles) return true;

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some(role => user?.roles?.includes(role));
  }
}
```

### Applying Guards

```typescript
// Method level
@UseGuards(AuthGuard)
@Get('profile')
getProfile() { ... }

// Controller level
@Controller('admin')
@UseGuards(AuthGuard, RolesGuard)
export class AdminController { ... }

// Global level (main.ts)
app.useGlobalGuards(new AuthGuard());

// Global with DI support
@Module({
  providers: [{ provide: APP_GUARD, useClass: AuthGuard }],
})
export class AppModule {}
```

### Guard Execution Order

Guards execute in this order:
1. Global guards (registration order)
2. Controller-level guards (left to right in `@UseGuards()`)
3. Method-level guards (left to right)

If any guard returns `false`, NestJS throws a `ForbiddenException` (403).

## Common Pitfalls

1. **Using middleware for authorization**: Middleware can't read route metadata. Use guards with `Reflector` for role-based access.
2. **Forgetting async guards**: If your guard calls a database, return `Promise<boolean>` or `Observable<boolean>`, not a synchronous value.
3. **Not handling missing metadata**: Always check if metadata exists (`if (!requiredRoles) return true`) to avoid blocking undecorated routes.

## Best Practices

- Use `APP_GUARD` provider token for global guards with DI support
- Combine `@SetMetadata()` decorators with guards for declarative authorization
- Return `false` for denied access (NestJS auto-throws ForbiddenException) or throw a custom exception for specific status codes
- Keep guards focused — one guard per concern (auth, roles, throttle)

## Summary

Guards implement the `CanActivate` interface to allow or deny request processing. They execute after middleware but before interceptors and pipes. Use `ExecutionContext` and `Reflector` to read route metadata for role-based authorization. Apply with `@UseGuards()` at method, controller, or global level.

## Code Examples

**A role-based guard using Reflector to read @Roles() metadata — returns true if the user has any of the required roles**

```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.getAllAndOverride<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!roles) return true;
    const { user } = context.switchToHttp().getRequest();
    return roles.some(role => user?.roles?.includes(role));
  }
}
```


## Resources

- [Guards Documentation](https://docs.nestjs.com/guards) — Official guide to implementing guards for authorization

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*