---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-lifecycle-events"
---

# Lifecycle Events

## Introduction

NestJS applications have a defined lifecycle—from initialization to shutdown. Lifecycle hooks let you run code at specific points: when modules initialize, when the app starts listening, or when it's shutting down. This is essential for resource management.

## Key Concepts

- **OnModuleInit**: Called after module dependencies are resolved
- **OnApplicationBootstrap**: Called after all modules initialized, before listening
- **OnModuleDestroy**: Called when app receives shutdown signal
- **BeforeApplicationShutdown**: Called before connections close

## Real World Context

Use lifecycle hooks for:
- Initializing database connections
- Starting background jobs
- Warming caches
- Gracefully closing connections on shutdown
- Cleanup operations

## Deep Dive

### Lifecycle Hook Order

1. `onModuleInit()` - Each provider, then each module
2. `onApplicationBootstrap()` - Same order
3. App starts listening for connections
4. `onModuleDestroy()` - On shutdown signal
5. `beforeApplicationShutdown()` - Same order
6. All connections closed
7. `onApplicationShutdown()` - Same order
8. Process exits

### Implementing Hooks

```typescript
import { Injectable, OnModuleInit, OnApplicationBootstrap, OnModuleDestroy } from '@nestjs/common';

@Injectable()
export class DatabaseService implements OnModuleInit, OnApplicationBootstrap, OnModuleDestroy {
  async onModuleInit() {
    console.log('Module dependencies resolved');
    await this.connect();
  }

  async onApplicationBootstrap() {
    console.log('App ready, about to listen');
    await this.runMigrations();
  }

  async onModuleDestroy() {
    console.log('Shutting down...');
    await this.disconnect();
  }
}
```

### Enabling Shutdown Hooks

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable graceful shutdown
  app.enableShutdownHooks();
  
  await app.listen(3000);
}
```

### BeforeApplicationShutdown

```typescript
@Injectable()
export class AppService implements BeforeApplicationShutdown {
  async beforeApplicationShutdown(signal?: string) {
    console.log(`Received signal: ${signal}`);
    // Cleanup before connections close
    await this.flushLogs();
  }
}
```

## Common Pitfalls

1. **Forgetting enableShutdownHooks**: Without it, shutdown hooks don't fire on SIGTERM/SIGINT.
2. **Blocking async operations**: Long operations in hooks delay startup/shutdown.
3. **Not handling errors**: Unhandled errors in hooks can crash the app.

## Best Practices

- Always enable shutdown hooks in production
- Keep hook operations fast; queue longer work
- Log hook execution for debugging
- Handle errors gracefully in hooks
- Use timeouts for shutdown operations

## Summary

Lifecycle hooks run code at specific points in the app lifecycle. Implement OnModuleInit for initialization, OnModuleDestroy for cleanup. Enable shutdown hooks with enableShutdownHooks() for graceful termination.

## Resources

- [Lifecycle Events](https://docs.nestjs.com/fundamentals/lifecycle-events) — Official lifecycle events guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*