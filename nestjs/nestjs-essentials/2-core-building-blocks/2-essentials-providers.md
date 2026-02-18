---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-providers"
---

# Providers & Dependency Injection

## Introduction

Dependency Injection (DI) is what makes NestJS applications testable, maintainable, and loosely coupled. Instead of classes creating their own dependencies, the framework **injects** them automatically. This is the foundation of enterprise-grade architecture.

## Key Concepts

- **Provider**: Any class that can be injected as a dependency (services, repositories, factories)
- **@Injectable()**: Decorator that marks a class as available for DI
- **Dependency Injection**: Design pattern where dependencies are provided rather than created
- **IoC Container**: The Nest runtime that manages instance creation and injection

## Real World Context

Without DI, you'd write:
```typescript
class UserController {
  private service = new UserService(new DatabaseConnection());
}
```

This is **tightly coupled**—testing requires a real database. With DI:
```typescript
class UserController {
  constructor(private userService: UserService) {}
}
```

Now you can inject a mock `UserService` in tests. This pattern enables unit testing, swapping implementations, and managing complex dependency graphs.

## Deep Dive

### Creating a Provider

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat): Cat {
    this.cats.push(cat);
    return cat;
  }

  findAll(): Cat[] {
    return this.cats;
  }

  findOne(id: string): Cat | undefined {
    return this.cats.find(cat => cat.id === id);
  }
}
```

### Injecting a Provider

Constructor injection is the standard approach:

```typescript
@Controller('cats')
export class CatsController {
  constructor(private readonly catsService: CatsService) {}

  @Get()
  findAll() {
    return this.catsService.findAll();
  }
}
```

### Registering Providers

Providers must be registered in a module:

```typescript
@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

### Provider Types

| Type | Use Case | Syntax |
|------|----------|--------|
| Standard | Most services | `providers: [CatsService]` |
| Value | Configuration, mocks | `{ provide: 'API_KEY', useValue: 'abc123' }` |
| Class | Conditional implementations | `{ provide: Logger, useClass: ProdLogger }` |
| Factory | Dynamic creation | `{ provide: 'DB', useFactory: () => ... }` |

### Injection Scopes

By default, providers are **singletons** (one instance per app). You can change this:

```typescript
@Injectable({ scope: Scope.REQUEST })
export class RequestScopedService {}
```

| Scope | Lifetime | Use Case |
|-------|----------|---------|
| DEFAULT | Singleton | Most services |
| REQUEST | Per HTTP request | Request-specific data |
| TRANSIENT | New instance each injection | Stateful helpers |

## Common Pitfalls

1. **Forgetting @Injectable()**: The decorator is required for TypeScript to emit metadata. Without it, injection silently fails.
2. **Not registering in module**: A provider must be in the `providers` array of a module to be injectable.
3. **Circular dependencies**: Service A injects B, B injects A. Use `forwardRef()` to resolve.

## Best Practices

- **Single Responsibility**: Each service handles one domain concern
- **Interface-based design**: Depend on abstractions, inject implementations
- **Favor constructor injection**: It's explicit and works with TypeScript types
- **Keep providers stateless when possible**: Easier to test and reason about

## Summary

Providers are injectable classes marked with `@Injectable()`. The NestJS IoC container creates and injects them automatically via constructor parameters. This enables loose coupling, testability, and clean architecture.

## Resources

- [Providers Documentation](https://docs.nestjs.com/providers) — Official guide to providers and dependency injection
- [Custom Providers](https://docs.nestjs.com/fundamentals/custom-providers) — Advanced provider patterns and techniques

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*