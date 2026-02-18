---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-versioning-strategies"
---

# Versioning Strategies

## Introduction

Choosing the right versioning strategy depends on your API consumers, change frequency, and organizational needs. Each approach has trade-offs between simplicity, flexibility, and visibility.

## Key Concepts

- **Major Versions**: Breaking changes (v1 → v2)
- **Deprecation**: Marking versions as end-of-life
- **Migration Path**: Helping clients upgrade
- **Sunset Header**: HTTP header indicating deprecation

## Real World Context

Versioning strategy decisions:
- Public API: URI versioning (visible, cacheable)
- Mobile app: Header versioning (clean URLs)
- Enterprise: Longer support windows
- Startup: Faster deprecation cycles

## Deep Dive

### Strategy 1: Separate Controllers

```typescript
// v1/users.controller.ts
@Controller({ path: 'users', version: '1' })
export class UsersV1Controller {
  constructor(private usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAllLegacy();
  }
}

// v2/users.controller.ts
@Controller({ path: 'users', version: '2' })
export class UsersV2Controller {
  constructor(private usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAllWithPagination();
  }
}
```

### Strategy 2: Shared Controller with Branching

```typescript
@Controller('users')
export class UsersController {
  @Get()
  @Version('1')
  findAllV1(): UserV1Dto[] {
    const users = this.usersService.findAll();
    return users.map(u => new UserV1Dto(u));
  }

  @Get()
  @Version('2')
  findAllV2(): UserV2Dto[] {
    const users = this.usersService.findAll();
    return users.map(u => new UserV2Dto(u));
  }
}
```

### Strategy 3: Version in Service Layer

```typescript
@Injectable()
export class UsersService {
  findAll(version: string): UserDto[] {
    const users = this.repository.findAll();
    
    if (version === '1') {
      return users.map(u => this.toV1Dto(u));
    }
    return users.map(u => this.toV2Dto(u));
  }
}

@Controller('users')
export class UsersController {
  @Get()
  findAll(@Headers('x-api-version') version: string = '1') {
    return this.usersService.findAll(version);
  }
}
```

### Deprecation with Sunset Header

```typescript
@Injectable()
export class DeprecationInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const response = context.switchToHttp().getResponse();
    const version = context.switchToHttp().getRequest().version;

    if (version === '1') {
      response.setHeader('Deprecation', 'true');
      response.setHeader('Sunset', 'Sat, 31 Dec 2025 23:59:59 GMT');
      response.setHeader('Link', '</api/v2/users>; rel="successor-version"');
    }

    return next.handle();
  }
}
```

### Versioning with Different DTOs

```typescript
// User remains same, DTOs differ per version
export class UserV1Dto {
  id: string;
  name: string;  // V1: single name field
}

export class UserV2Dto {
  id: string;
  firstName: string;  // V2: split name fields
  lastName: string;
  fullName: string;   // V2: computed field
}
```

### Module Organization

```
src/
├── users/
│   ├── v1/
│   │   ├── users.controller.ts
│   │   └── dto/
│   ├── v2/
│   │   ├── users.controller.ts
│   │   └── dto/
│   ├── users.service.ts     # Shared service
│   └── users.module.ts
```

### Version Negotiation

```typescript
@Injectable()
export class VersionNegotiationMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Check header first, then query, then default
    const version = 
      req.headers['x-api-version'] ||
      req.query.version ||
      '2'; // Latest as default

    req['version'] = version;
    next();
  }
}
```

## Common Pitfalls

1. **Version proliferation**: Too many versions increase maintenance.
2. **Unclear deprecation**: Clients need clear timelines.
3. **Inconsistent versioning**: Apply same strategy everywhere.

## Best Practices

- Document version differences clearly
- Use Sunset headers for deprecation notices
- Provide migration guides
- Support N-1 version minimum
- Set clear end-of-life dates

## Summary

Versioning strategies range from separate controllers to shared services with branching. Use deprecation headers to notify clients, organize code by version, and maintain clear migration paths. Choose strategy based on change frequency and client needs.

## Resources

- [Versioning](https://docs.nestjs.com/techniques/versioning) — Versioning techniques

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*