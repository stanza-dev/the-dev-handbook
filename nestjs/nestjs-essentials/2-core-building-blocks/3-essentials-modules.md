---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-modules"
---

# Modules

## Introduction

Modules are the organizational backbone of NestJS applications. They group related functionality together, encapsulate implementation details, and define clear boundaries between different parts of your application. Think of modules as self-contained features that can be composed to build complex systems.

## Key Concepts

- **Module**: A class decorated with `@Module()` that organizes providers, controllers, and imports
- **Root Module**: The entry point (`AppModule`) that imports all other modules
- **Feature Module**: A module encapsulating a specific domain (e.g., `UsersModule`)
- **Application Graph**: The dependency tree NestJS builds from module relationships

## Real World Context

In a production e-commerce application:
- `UsersModule` handles authentication and user management
- `ProductsModule` manages the product catalog
- `OrdersModule` processes orders and payments
- `SharedModule` provides common utilities (logging, caching)

Each module can be developed, tested, and maintained independently. Teams can own specific modules without conflicting with others.

## Deep Dive

### Module Structure

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  imports: [],        // Other modules this module depends on
  controllers: [CatsController],  // Controllers to instantiate
  providers: [CatsService],       // Services available within this module
  exports: [CatsService],         // Services available to importing modules
})
export class CatsModule {}
```

### @Module() Properties

| Property | Purpose | Example |
|----------|---------|--------|
| `imports` | Modules whose exported providers are needed here | `[DatabaseModule, ConfigModule]` |
| `controllers` | Controllers that handle routes for this module | `[CatsController]` |
| `providers` | Services instantiated and available within module | `[CatsService, CatsRepository]` |
| `exports` | Providers to expose to other modules | `[CatsService]` |

### Module Composition

```typescript
// Root module imports feature modules
@Module({
  imports: [
    ConfigModule.forRoot(),
    DatabaseModule,
    UsersModule,
    ProductsModule,
    OrdersModule,
  ],
})
export class AppModule {}
```

### Sharing Providers Between Modules

By default, providers are **private** to their module. To share:

1. **Export** from the source module:
```typescript
@Module({
  providers: [CatsService],
  exports: [CatsService], // Now available to importers
})
export class CatsModule {}
```

2. **Import** in the consuming module:
```typescript
@Module({
  imports: [CatsModule], // CatsService now injectable here
})
export class DogsModule {}
```

### Global Modules

For utilities needed everywhere (logging, config):

```typescript
@Global()
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule {}
```

## Common Pitfalls

1. **Forgetting to export**: A provider in `providers` isn't automatically available elsewhere. You must add it to `exports`.
2. **Circular imports**: Module A imports B, B imports A. Use `forwardRef(() => ModuleA)` to resolve.
3. **Over-using @Global()**: Global modules reduce encapsulation. Use sparingly for true cross-cutting concerns.

## Best Practices

- **One module per feature**: `UsersModule`, `ProductsModule`—not `ServicesModule`
- **Keep modules focused**: If a module has 10+ providers, consider splitting
- **Re-export for convenience**: A `DatabaseModule` might re-export `TypeOrmModule`
- **Use barrel exports**: `export * from './cats.service'` in `index.ts`

## Summary

Modules organize NestJS applications into cohesive blocks. The `@Module()` decorator defines imports (dependencies), controllers, providers, and exports. This encapsulation enables scalable architecture where teams can work independently on feature modules.

## Resources

- [Modules Documentation](https://docs.nestjs.com/modules) — Official guide to creating and organizing modules

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*