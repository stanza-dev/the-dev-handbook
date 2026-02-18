---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-module-reference"
---

# Module Reference (ModuleRef)

## Introduction

Sometimes you need to dynamically resolve providers at runtime rather than through constructor injection. ModuleRef provides programmatic access to the DI container, enabling dynamic instantiation and lookup.

## Key Concepts

- **ModuleRef**: Service for dynamic provider resolution
- **get()**: Retrieve a provider instance
- **resolve()**: Create new instance (for transient/request scoped)
- **Lazy Loading**: Load modules on demand

## Deep Dive

### Basic ModuleRef Usage

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import { ModuleRef } from '@nestjs/core';

@Injectable()
export class DynamicService implements OnModuleInit {
  private usersService: UsersService;

  constructor(private moduleRef: ModuleRef) {}

  onModuleInit() {
    this.usersService = this.moduleRef.get(UsersService);
  }
}
```

### Resolving Transient Providers

```typescript
// For transient/request-scoped providers
const instance = await this.moduleRef.resolve(TransientService);
```

### Getting Provider from Different Context

```typescript
// strict: false allows looking outside current module
const service = this.moduleRef.get(SomeService, { strict: false });
```

### Creating Instances Dynamically

```typescript
@Injectable()
export class PluginLoader {
  constructor(private moduleRef: ModuleRef) {}

  async loadPlugin(pluginClass: Type<Plugin>): Promise<Plugin> {
    return this.moduleRef.create(pluginClass);
  }
}
```

### Lazy Loading Modules

```typescript
import { LazyModuleLoader } from '@nestjs/core';

@Injectable()
export class FeatureService {
  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  async loadReporting() {
    const { ReportingModule } = await import('./reporting.module');
    const moduleRef = await this.lazyModuleLoader.load(() => ReportingModule);
    return moduleRef.get(ReportingService);
  }
}
```

## Common Pitfalls

1. **Using before initialization**: ModuleRef isn't available in constructor. Use OnModuleInit.
2. **strict: true default**: By default, get() only looks in current module context.
3. **Overusing ModuleRef**: Prefer constructor injection. Use ModuleRef for truly dynamic cases.

## Best Practices

- Prefer constructor injection for static dependencies
- Use ModuleRef for plugins, dynamic loading, circular dependency breaking
- Access in onModuleInit, not constructor
- Document why you're using dynamic resolution

## Summary

ModuleRef provides runtime access to the DI container. Use get() for singletons, resolve() for scoped providers, and create() for dynamic instantiation. Prefer constructor injection; use ModuleRef only when needed.

## Resources

- [Module Reference](https://docs.nestjs.com/fundamentals/module-ref) — Official module reference guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*