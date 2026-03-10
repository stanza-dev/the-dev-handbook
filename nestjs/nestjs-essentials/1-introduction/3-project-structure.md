---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-project-structure"
---

# Project Structure

## Introduction

A well-organized project structure is the difference between a maintainable codebase and a tangled mess. NestJS provides conventions that scale from small APIs to large enterprise applications with multiple teams.

## Key Concepts

- **Module-per-feature**: Each domain feature gets its own module folder
- **Barrel exports**: `index.ts` files that re-export module contents
- **Separation of concerns**: Controllers handle HTTP, services handle business logic
- **Co-location**: Keep related files (controller, service, DTOs) together

## Real World Context

When your application grows to 50+ endpoints across multiple domains (users, products, orders, payments), structure becomes critical. Teams can work independently on different modules without stepping on each other's toes. Code reviews become easier when reviewers know exactly where to look.

## Deep Dive

### Standard Project Layout

```
src/
├── main.ts                 # Application entry point
├── app.module.ts           # Root module
├── common/                 # Shared utilities
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   └── pipes/
├── config/                 # Configuration module
│   ├── config.module.ts
│   └── config.service.ts
├── users/                  # Feature module
│   ├── users.module.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts
│   ├── entities/
│   │   └── user.entity.ts
│   └── users.repository.ts
└── products/               # Another feature module
    └── ...
```

### Key Files Explained

| File | Purpose |
|------|--------|
| `main.ts` | Bootstrap function, global middleware, CORS, validation |
| `app.module.ts` | Root module that imports all feature modules |
| `*.module.ts` | Declares providers, controllers, imports, exports |
| `*.controller.ts` | HTTP route handlers |
| `*.service.ts` | Business logic, database operations |
| `*.dto.ts` | Data Transfer Objects for request/response typing |
| `*.entity.ts` | Database model definitions |

### Monorepo Structure (Advanced)

For larger projects, NestJS supports monorepos:

```
apps/
├── api/
├── admin-api/
└── worker/
libs/
├── shared/
├── database/
└── auth/
```

## Common Pitfalls

1. **Flat structure**: Dumping all files in `src/` becomes unmanageable past 10 files. Always use feature folders.
2. **Circular module imports**: Module A imports B, B imports A. Use `forwardRef()` or restructure to avoid.
3. **Business logic in controllers**: Controllers should be thin—delegate to services.

## Best Practices

- **One module per feature domain**: `UsersModule`, `ProductsModule`, `OrdersModule`
- **Shared code in `common/` or `shared/`**: Guards, interceptors, decorators used across modules
- **DTOs for every endpoint**: Type safety and validation at the boundary
- **Use the CLI**: `nest g resource users` creates the entire folder structure

## Summary

NestJS project structure follows a module-per-feature pattern where each domain (users, products) has its own folder containing the module, controller, service, and related files. This organization scales well and makes navigation intuitive for new team members.

## Code Examples

**Root module composition — each feature domain has its own module, imported into the central AppModule**

```typescript
// app.module.ts — Root module importing feature modules
import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';
import { ProductsModule } from './products/products.module';
import { ConfigModule } from './config/config.module';

@Module({
  imports: [ConfigModule.forRoot(), UsersModule, ProductsModule],
})
export class AppModule {}
```


## Resources

- [Modules Documentation](https://docs.nestjs.com/modules) — Official guide to organizing code with modules
- [CLI Monorepo Mode](https://docs.nestjs.com/cli/monorepo) — Setting up monorepo projects with Nest CLI

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*