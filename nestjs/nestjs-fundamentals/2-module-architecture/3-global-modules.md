---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-global-modules"
---

# Global Modules

## Introduction

Some modules are needed everywhere—configuration, logging, caching. Instead of importing them in every module, you can make them global. Global modules are available application-wide without explicit imports.

## Key Concepts

- **@Global()**: Decorator that makes a module globally available
- **Root Import**: Global modules still need one import (typically in AppModule)
- **Provider Scope**: Global affects module scope, not provider instances

## Deep Dive

### Creating a Global Module

```typescript
import { Global, Module } from '@nestjs/common';

@Global()
@Module({
  providers: [ConfigService, LoggerService],
  exports: [ConfigService, LoggerService],
})
export class CoreModule {}
```

### Root Import

```typescript
@Module({
  imports: [CoreModule], // Import once in AppModule
})
export class AppModule {}
```

### Using Global Providers

```typescript
// No need to import CoreModule
@Module({
  providers: [UsersService],
})
export class UsersModule {}

// ConfigService is available
@Injectable()
export class UsersService {
  constructor(private configService: ConfigService) {}
}
```

### When to Use Global Modules

- Configuration services
- Logging services
- Database connections
- Event emitters
- Caching services

## Common Pitfalls

1. **Overusing global**: Not everything should be global. It reduces encapsulation.
2. **Forgetting to export**: @Global() makes the module available, but you still need exports.
3. **Singleton confusion**: Global doesn't change scope. Providers are still singletons by default.

## Best Practices

- Use global sparingly for truly cross-cutting concerns
- Document which modules are global
- Prefer explicit imports for better dependency tracking
- Consider lazy loading for large applications

## Summary

@Global() makes modules available everywhere without explicit imports. Use it for cross-cutting concerns like configuration and logging. Don't overuse—explicit imports improve code clarity.

## Resources

- [Modules](https://docs.nestjs.com/modules#global-modules) — Global modules documentation

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*