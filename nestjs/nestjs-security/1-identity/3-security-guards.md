---
source_course: "nestjs-security"
source_lesson: "nestjs-security-guards"
---

# Guards Deep Dive

## Introduction

Guards are the gatekeepers of your NestJS application. They determine whether a request should be handled by the route handler based on certain conditions (permissions, roles, ACLs, etc.). Understanding guards is essential for implementing any authentication or authorization strategy.

## Key Concepts

- **Guard**: A class implementing `CanActivate` interface
- **ExecutionContext**: Provides access to request details and route metadata
- **CanActivate**: Interface requiring a method that returns boolean or Promise<boolean>
- **APP_GUARD**: Token for registering global guards via DI

## Real World Context

Guards handle security decisions:
- Is the user authenticated?
- Does the user have the required role?
- Is the API key valid?
- Has the rate limit been exceeded?

## Deep Dive

### Basic Guard Structure

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return this.validateRequest(request);
  }

  private validateRequest(request: any): boolean {
    // Your validation logic
    return true;
  }
}
```

### ExecutionContext Methods

```typescript
// Get request/response objects
const request = context.switchToHttp().getRequest();
const response = context.switchToHttp().getResponse();

// Get handler and class info
const handler = context.getHandler(); // Route handler method
const controller = context.getClass(); // Controller class

// Get request type (http, ws, rpc)
const type = context.getType(); // 'http' | 'ws' | 'rpc'
```

### Reading Metadata

```typescript
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Read from handler first, fallback to class
    const roles = this.reflector.getAllAndOverride<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (!roles) return true;
    
    const { user } = context.switchToHttp().getRequest();
    return roles.some(role => user.roles?.includes(role));
  }
}
```

### Guard Execution Order

1. Global guards (in registration order)
2. Controller guards (in decorator order)
3. Method guards (in decorator order)

If any guard returns false, the request is rejected with a 403 Forbidden.

### Async Guards

```typescript
@Injectable()
export class ApiKeyGuard implements CanActivate {
  constructor(private apiKeysService: ApiKeysService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const apiKey = request.headers['x-api-key'];
    
    if (!apiKey) return false;
    
    const isValid = await this.apiKeysService.validate(apiKey);
    return isValid;
  }
}
```

## Common Pitfalls

1. **Throwing vs returning false**: Throwing an exception gives you control over the error response. Returning false gives a generic 403.
2. **Not using DI**: Instantiating guards manually (`new Guard()`) loses DI. Use `APP_GUARD` for global guards needing dependencies.
3. **Heavy operations in guards**: Guards run on every request. Cache expensive lookups.

## Best Practices

- Throw specific exceptions for clear error messages
- Use the Reflector for reading custom decorator metadata
- Keep guards focused on one responsibility
- Cache database lookups when possible
- Use composition for complex guard logic

## Summary

Guards implement `CanActivate` and return boolean values to allow or deny requests. They access request context via `ExecutionContext` and read metadata via `Reflector`. Apply guards at method, controller, or global level.

## Resources

- [Guards Documentation](https://docs.nestjs.com/guards) — Official guide to guards

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*