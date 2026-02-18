---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-decorator-composition"
---

# Decorator Composition Patterns

## Introduction

Real-world applications require sophisticated decorator patterns. Combining decorators strategically reduces boilerplate, enforces consistency, and creates expressive APIs. Learn the patterns that scale from small projects to enterprise applications.

## Key Concepts

- **Composition**: Building complex decorators from simple ones
- **Domain Decorators**: Decorators that express business concepts
- **Decorator Inheritance**: Extending existing decorator behavior
- **Decorator Testing**: Verifying decorator behavior works correctly

## Real World Context

Enterprise NestJS applications use composition for:
- API versioning with consistent error handling
- Multi-tenant route protection
- Feature flags and A/B testing
- Audit logging and compliance

## Deep Dive

### Domain-Specific API Decorator

```typescript
import { applyDecorators, Get, HttpCode, HttpStatus } from '@nestjs/common';
import { ApiOperation, ApiResponse, ApiParam } from '@nestjs/swagger';

export function GetById(resourceName: string) {
  return applyDecorators(
    Get(':id'),
    HttpCode(HttpStatus.OK),
    ApiOperation({ summary: `Get ${resourceName} by ID` }),
    ApiParam({ name: 'id', description: `${resourceName} ID` }),
    ApiResponse({ status: 200, description: `${resourceName} found` }),
    ApiResponse({ status: 404, description: `${resourceName} not found` }),
  );
}

// Usage
@GetById('User')
findOne(@Param('id') id: string) { ... }
```

### Authenticated Endpoint Decorator

```typescript
export function AuthenticatedEndpoint(
  method: 'GET' | 'POST' | 'PUT' | 'DELETE',
  path: string,
  options?: { roles?: string[]; rateLimit?: number },
) {
  const decorators = [
    UseGuards(AuthGuard),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Invalid or missing token' }),
  ];

  // Add method decorator
  switch (method) {
    case 'GET': decorators.push(Get(path)); break;
    case 'POST': decorators.push(Post(path)); break;
    case 'PUT': decorators.push(Put(path)); break;
    case 'DELETE': decorators.push(Delete(path)); break;
  }

  // Add optional guards
  if (options?.roles?.length) {
    decorators.push(SetMetadata('roles', options.roles));
    decorators.push(UseGuards(RolesGuard));
  }

  if (options?.rateLimit) {
    decorators.push(Throttle({ default: { limit: options.rateLimit, ttl: 60000 } }));
  }

  return applyDecorators(...decorators);
}

// Usage
@AuthenticatedEndpoint('POST', 'users', { roles: ['admin'], rateLimit: 10 })
createUser(@Body() dto: CreateUserDto) { ... }
```

### Feature Flag Decorator

```typescript
export const FEATURE_FLAG_KEY = 'featureFlag';

export function FeatureFlag(flag: string) {
  return applyDecorators(
    SetMetadata(FEATURE_FLAG_KEY, flag),
    UseGuards(FeatureFlagGuard),
  );
}

@Injectable()
export class FeatureFlagGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    private featureService: FeatureService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const flag = this.reflector.get<string>(FEATURE_FLAG_KEY, context.getHandler());
    if (!flag) return true;
    
    const request = context.switchToHttp().getRequest();
    return this.featureService.isEnabled(flag, request.user);
  }
}

// Usage
@FeatureFlag('new-checkout-flow')
@Post('checkout')
checkout() { ... }
```

### Audit Logging Decorator

```typescript
export function Audit(action: string) {
  return applyDecorators(
    SetMetadata('audit:action', action),
    UseInterceptors(AuditInterceptor),
  );
}

@Injectable()
export class AuditInterceptor implements NestInterceptor {
  constructor(
    private reflector: Reflector,
    private auditService: AuditService,
  ) {}

  intercept(context: ExecutionContext, next: CallHandler) {
    const action = this.reflector.get<string>('audit:action', context.getHandler());
    const request = context.switchToHttp().getRequest();

    return next.handle().pipe(
      tap(() => {
        this.auditService.log({
          action,
          userId: request.user?.id,
          resource: request.url,
          timestamp: new Date(),
        });
      }),
    );
  }
}

// Usage
@Audit('USER_DELETE')
@Delete(':id')
deleteUser(@Param('id') id: string) { ... }
```

### Testing Decorators

```typescript
describe('RolesGuard', () => {
  it('should allow access when user has required role', async () => {
    const module = await Test.createTestingModule({
      controllers: [TestController],
      providers: [RolesGuard],
    }).compile();

    const guard = module.get<RolesGuard>(RolesGuard);
    const mockContext = createMockExecutionContext({
      user: { roles: ['admin'] },
      handler: TestController.prototype.adminOnly,
    });

    expect(guard.canActivate(mockContext)).toBe(true);
  });
});
```

## Common Pitfalls

1. **Over-composition**: Too many decorators in one makes debugging hard. Keep compositions focused.
2. **Hidden dependencies**: Composite decorators may require certain providers. Document requirements.
3. **Order sensitivity**: Some decorators must come before others. Test and document order.

## Best Practices

- Create domain-specific decorators that express business intent
- Document what each composite decorator includes
- Test guards and interceptors that read decorator metadata
- Use consistent naming conventions (@Auth, @Audit, @Feature)

## Summary

Decorator composition creates expressive, domain-specific APIs. Use `applyDecorators()` to combine multiple decorators. Create guards and interceptors that read metadata. Document composite decorators and test the guards/interceptors that implement their behavior.

## Resources

- [Decorator Composition](https://docs.nestjs.com/custom-decorators#decorator-composition) — Composing decorators with applyDecorators

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*