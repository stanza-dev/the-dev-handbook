---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-dynamic-modules"
---

# Dynamic Modules

## Introduction

Static modules have fixed configuration. But what if you need to pass options when importing? Dynamic modules accept configuration, making them reusable across different contexts with different settings.

## Key Concepts

- **Dynamic Module**: Module with configurable options via static methods
- **forRoot()**: Configure once for root module (global settings)
- **forFeature()**: Configure per-feature module (specific settings)
- **Async Configuration**: Factory-based configuration with dependency injection

## Real World Context

Consider a database module. Different projects use different databases, credentials, and settings. A dynamic module accepts these as options rather than hardcoding them.

## Deep Dive

### Basic Dynamic Module

```typescript
import { Module, DynamicModule } from '@nestjs/common';

@Module({})
export class ConfigModule {
  static forRoot(options: ConfigOptions): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'CONFIG_OPTIONS',
          useValue: options,
        },
        ConfigService,
      ],
      exports: [ConfigService],
    };
  }
}
```

### Using the Dynamic Module

```typescript
@Module({
  imports: [
    ConfigModule.forRoot({
      folder: './config',
      envFile: '.env',
    }),
  ],
})
export class AppModule {}
```

### forRoot vs forFeature Pattern

```typescript
@Module({})
export class DatabaseModule {
  // Global configuration (call once)
  static forRoot(options: DatabaseOptions): DynamicModule {
    return {
      module: DatabaseModule,
      global: true,
      providers: [/* connection providers */],
      exports: [/* connection */],
    };
  }

  // Feature-specific configuration (call per module)
  static forFeature(entities: Entity[]): DynamicModule {
    return {
      module: DatabaseModule,
      providers: [/* repository providers for entities */],
      exports: [/* repositories */],
    };
  }
}
```

### Async Configuration

```typescript
@Module({})
export class DatabaseModule {
  static forRootAsync(options: AsyncModuleOptions): DynamicModule {
    return {
      module: DatabaseModule,
      imports: options.imports || [],
      providers: [
        {
          provide: 'DB_OPTIONS',
          useFactory: options.useFactory,
          inject: options.inject || [],
        },
        DatabaseService,
      ],
      exports: [DatabaseService],
    };
  }
}

// Usage
DatabaseModule.forRootAsync({
  imports: [ConfigModule],
  useFactory: (config: ConfigService) => ({
    host: config.get('DB_HOST'),
    port: config.get('DB_PORT'),
  }),
  inject: [ConfigService],
})
```

## Common Pitfalls

1. **Forgetting to return DynamicModule**: Static methods must return the full module definition.
2. **Not exporting providers**: Remember to export providers that other modules need.
3. **Missing global: true for forRoot**: Root modules are often global—don't forget this.

## Best Practices

- Use forRoot for global, one-time configuration
- Use forFeature for module-specific configuration
- Provide forRootAsync for factory-based configuration
- Document expected options with TypeScript interfaces

## Summary

Dynamic modules accept configuration via static methods like forRoot() and forFeature(). They return DynamicModule objects with providers and exports. Use forRootAsync for configuration that depends on other services.

## Resources

- [Dynamic Modules](https://docs.nestjs.com/fundamentals/dynamic-modules) — Official dynamic modules guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*