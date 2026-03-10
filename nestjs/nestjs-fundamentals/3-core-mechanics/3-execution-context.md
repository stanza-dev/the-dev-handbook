---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-execution-context"
---

# Execution Context

## Introduction

Guards, interceptors, and filters need information about the current request—but requests can be HTTP, WebSocket, or microservice. ExecutionContext provides a unified API to access request metadata regardless of the transport layer.

## Key Concepts

- **ExecutionContext**: Extends ArgumentsHost with handler metadata
- **ArgumentsHost**: Provides access to handler arguments (req, res, next)
- **switchToHttp()**: Get HTTP-specific context
- **getHandler()/getClass()**: Get route handler metadata

## Real World Context

Cross-cutting concerns like authentication, rate limiting, and logging need to inspect requests regardless of the transport layer. ExecutionContext provides a unified API to access request data across HTTP, WebSocket, and microservice contexts — so a single `AuthGuard` can protect REST endpoints, GraphQL resolvers, and WebSocket gateways.

## Deep Dive

### ArgumentsHost Basics

```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const type = host.getType(); // 'http' | 'rpc' | 'ws'
    
    if (type === 'http') {
      const ctx = host.switchToHttp();
      const request = ctx.getRequest<Request>();
      const response = ctx.getResponse<Response>();
    }
  }
}
```

### ExecutionContext in Guards

```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get metadata from decorator
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    
    // Get handler and class info
    const handler = context.getHandler(); // Method reference
    const className = context.getClass().name; // Controller class name
    
    // Access request
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    return roles.some(role => user.roles?.includes(role));
  }
}
```

### Transport-Agnostic Code

```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const type = context.getType();
    
    if (type === 'http') {
      const req = context.switchToHttp().getRequest();
      console.log(`HTTP ${req.method} ${req.url}`);
    } else if (type === 'rpc') {
      const data = context.switchToRpc().getData();
      console.log('RPC call', data);
    } else if (type === 'ws') {
      const client = context.switchToWs().getClient();
      console.log('WS connection', client.id);
    }
    
    return next.handle();
  }
}
```

### Reading Custom Metadata

```typescript
// Custom decorator
export const Public = () => SetMetadata('isPublic', true);

// In guard
const isPublic = this.reflector.getAllAndOverride<boolean>('isPublic', [
  context.getHandler(),
  context.getClass(),
]);
```

## Common Pitfalls

1. **Assuming HTTP**: Always check getType() before using switchToHttp().
2. **Wrong Reflector method**: Use getAllAndOverride() to check handler then class.
3. **Type assertions**: Use generics (getRequest<Request>()) for type safety.

## Best Practices

- Write transport-agnostic code when possible
- Use Reflector for custom metadata
- Type your requests and responses
- Check context type before accessing specific properties

## Summary

ExecutionContext provides unified access to request context across transports. Use switchToHttp(), switchToRpc(), or switchToWs() for transport-specific data. Access handler metadata with getHandler(), getClass(), and Reflector.

## Code Examples

**ExecutionContext in a guard — getHandler() reads route metadata, switchToHttp().getRequest() accesses the HTTP request**

```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    if (!roles) return true;
    const request = context.switchToHttp().getRequest();
    return roles.some(role => request.user?.roles?.includes(role));
  }
}
```


## Resources

- [Execution Context](https://docs.nestjs.com/fundamentals/execution-context) — Official execution context guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*