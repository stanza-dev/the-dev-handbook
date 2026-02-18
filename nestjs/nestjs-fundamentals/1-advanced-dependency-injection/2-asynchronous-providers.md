---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-asynchronous-providers"
---

# Asynchronous Providers

## Introduction

Some resources take time to initialize—database connections, remote configuration fetches, or service discovery. NestJS waits for async providers to resolve before starting the application, ensuring dependencies are ready when needed.

## Key Concepts

- **Async Provider**: Provider whose factory returns a Promise
- **Application Bootstrap**: Process of starting the NestJS app
- **Blocking Initialization**: App waits for async providers before accepting requests

## Real World Context

Consider connecting to a database. You can't serve requests until the connection is established. Async providers ensure the connection is ready before the app starts handling traffic.

## Deep Dive

### Async Factory Provider

```typescript
const databaseProvider = {
  provide: 'DATABASE_CONNECTION',
  useFactory: async (configService: ConfigService) => {
    const options = configService.get('database');
    const connection = await createConnection(options);
    console.log('Database connected');
    return connection;
  },
  inject: [ConfigService],
};
```

### With External API

```typescript
const configProvider = {
  provide: 'REMOTE_CONFIG',
  useFactory: async () => {
    const response = await fetch('https://config-server.example.com/app-config');
    return response.json();
  },
};
```

### Module with Async Configuration

```typescript
@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DB_HOST'),
        port: configService.get('DB_PORT'),
        database: configService.get('DB_NAME'),
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

### Handling Errors

```typescript
const provider = {
  provide: 'CRITICAL_SERVICE',
  useFactory: async () => {
    try {
      return await initializeCriticalService();
    } catch (error) {
      console.error('Failed to initialize:', error);
      throw error; // Prevents app from starting
    }
  },
};
```

## Common Pitfalls

1. **No error handling**: Unhandled rejections crash the app. Add try/catch.
2. **Slow initialization**: Long async operations delay startup. Consider timeouts.
3. **Not logging failures**: Silent failures make debugging impossible.

## Best Practices

- Add timeouts for external service connections
- Log initialization progress for debugging
- Gracefully handle failures where possible
- Consider health checks after initialization

## Summary

Async providers use factory functions returning Promises. NestJS waits for all async providers to resolve before accepting requests. Handle errors gracefully and add timeouts to prevent indefinite startup delays.

## Resources

- [Async Providers](https://docs.nestjs.com/fundamentals/async-providers) — Official async providers guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*