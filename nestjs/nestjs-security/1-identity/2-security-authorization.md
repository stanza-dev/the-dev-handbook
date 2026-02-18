---
source_course: "nestjs-security"
source_lesson: "nestjs-security-authorization"
---

# Authorization (RBAC)

## Introduction

Authorization answers "What can you do?" While authentication verifies identity, authorization controls access to resources. Role-Based Access Control (RBAC) is the most common pattern—users have roles, roles have permissions.

## Key Concepts

- **Authorization**: Determining what actions a user can perform
- **RBAC**: Role-Based Access Control—permissions assigned to roles
- **Guard**: NestJS mechanism that determines if a request can proceed
- **Reflector**: Service to read metadata set by decorators
- **Custom Decorator**: Reusable metadata attachments for routes

## Real World Context

Consider an e-commerce API:
- **Users** can view products and their own orders
- **Admins** can manage products and view all orders
- **Super Admins** can manage users and system settings

RBAC simplifies this by assigning permissions to roles rather than individual users.

## Deep Dive

### Define Roles

```typescript
export enum Role {
  User = 'user',
  Admin = 'admin',
  SuperAdmin = 'super_admin',
}
```

### Roles Decorator

```typescript
import { Reflector } from '@nestjs/core';

export const Roles = Reflector.createDecorator<Role[]>();
```

### Roles Guard

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { Role, Roles } from './roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get(Roles, context.getHandler());
    
    if (!requiredRoles) {
      return true; // No roles required, allow access
    }
    
    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some((role) => user.roles?.includes(role));
  }
}
```

### Using the Roles Decorator

```typescript
@Controller('admin')
@UseGuards(AuthGuard, RolesGuard)
export class AdminController {
  @Post('products')
  @Roles([Role.Admin, Role.SuperAdmin])
  createProduct(@Body() dto: CreateProductDto) {
    return this.productsService.create(dto);
  }

  @Delete('users/:id')
  @Roles([Role.SuperAdmin])
  deleteUser(@Param('id') id: string) {
    return this.usersService.delete(id);
  }
}
```

### Combining Guards

Guards execute in order. Typically:
1. AuthGuard (verify identity)
2. RolesGuard (check permissions)

```typescript
@UseGuards(AuthGuard, RolesGuard)
@Controller('admin')
export class AdminController {}
```

### Global Guard Registration

```typescript
// main.ts
app.useGlobalGuards(new AuthGuard(), new RolesGuard());

// Or via module for DI support
@Module({
  providers: [
    { provide: APP_GUARD, useClass: AuthGuard },
    { provide: APP_GUARD, useClass: RolesGuard },
  ],
})
export class AppModule {}
```

## Common Pitfalls

1. **Guard order matters**: AuthGuard must run before RolesGuard. If roles guard runs first, `user` won't exist on the request.
2. **Forgetting to attach roles to user**: The JWT payload must include the user's roles, or fetch them from database in the guard.
3. **Overly complex permissions**: Start with simple RBAC. Only add attribute-based access control (ABAC) when truly needed.

## Best Practices

- Keep role definitions in a central enum or constant file
- Use descriptive role names that match business terminology
- Implement the principle of least privilege
- Log authorization failures for security auditing
- Consider using CASL for complex authorization requirements

## Summary

RBAC in NestJS uses custom decorators to attach role requirements to routes and Guards to enforce them. The Reflector service reads decorator metadata. Combine AuthGuard and RolesGuard for complete access control.

## Resources

- [Authorization Documentation](https://docs.nestjs.com/security/authorization) — Official guide to authorization
- [CASL Integration](https://docs.nestjs.com/security/authorization#integrating-casl) — Advanced authorization with CASL

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*