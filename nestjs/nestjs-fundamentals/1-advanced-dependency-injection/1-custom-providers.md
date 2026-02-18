---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-custom-providers"
---

# Custom Providers

## Introduction

Standard providers use class names for injection tokens. But what about configuration values, factory-created instances, or swapping implementations for testing? Custom providers give you complete control over how dependencies are created and resolved.

## Key Concepts

- **Injection Token**: Identifier used to lookup a provider (class or string)
- **useValue**: Provide a fixed value (objects, primitives, mocks)
- **useClass**: Dynamically select which class to instantiate
- **useFactory**: Create providers with custom logic and dependencies

## Real World Context

Custom providers solve real problems:
- Inject configuration objects
- Mock services in tests
- Switch implementations based on environment
- Create complex objects with dependencies

## Deep Dive

### Value Providers

```typescript
// Inject a configuration object
const configProvider = {
  provide: 'CONFIG',
  useValue: {
    apiUrl: 'https://api.example.com',
    timeout: 5000,
  },
};

@Module({
  providers: [configProvider],
})
export class AppModule {}

// Inject with token
constructor(@Inject('CONFIG') private config: ConfigType) {}
```

### Class Providers

```typescript
// Environment-based implementation
const loggerProvider = {
  provide: LoggerService,
  useClass: process.env.NODE_ENV === 'production'
    ? ProductionLoggerService
    : DevelopmentLoggerService,
};
```

### Factory Providers

```typescript
const databaseProvider = {
  provide: 'DATABASE_CONNECTION',
  useFactory: async (configService: ConfigService) => {
    const config = configService.get('database');
    return createConnection(config);
  },
  inject: [ConfigService], // Dependencies for factory
};
```

### Async Factory Providers

```typescript
const asyncProvider = {
  provide: 'ASYNC_DATA',
  useFactory: async () => {
    const data = await fetchRemoteConfig();
    return data;
  },
};
```

## Common Pitfalls

1. **Forgetting inject array**: Factory providers need dependencies listed in `inject`.
2. **Circular string tokens**: Use Symbol or class references to avoid token collisions.
3. **Async providers blocking startup**: Long async operations delay app startup.

## Best Practices

- Use class references over string tokens when possible
- Create constants for string tokens to avoid typos
- Keep factory logic simple—delegate to services
- Document custom providers for team clarity

## Summary

Custom providers control how dependencies are created. Use `useValue` for constants, `useClass` for conditional implementations, and `useFactory` for complex creation logic. Inject custom providers using `@Inject(TOKEN)`.

## Resources

- [Custom Providers](https://docs.nestjs.com/fundamentals/custom-providers) — Official custom providers guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*