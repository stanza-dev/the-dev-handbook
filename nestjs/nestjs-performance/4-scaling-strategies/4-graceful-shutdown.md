---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-graceful-shutdown"
---

# Graceful Shutdown

## Introduction

Abrupt shutdowns lose in-flight requests and corrupt data. Graceful shutdown completes pending work before terminating, ensuring data integrity and user experience during deployments.

## Key Concepts

- **SIGTERM**: Signal to terminate gracefully
- **SIGINT**: Interrupt signal (Ctrl+C)
- **Drain Period**: Time to complete in-flight requests
- **Shutdown Hooks**: Cleanup callbacks

## Real World Context

Graceful shutdown prevents:
- Lost HTTP responses
- Incomplete database transactions
- Orphaned queue jobs
- Connection leaks

## Deep Dive

### Enable Shutdown Hooks

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Enable lifecycle hooks
  app.enableShutdownHooks();

  await app.listen(3000);
}
bootstrap();
```

### Implementing Shutdown Logic

```typescript
import { Injectable, OnModuleDestroy, BeforeApplicationShutdown, OnApplicationShutdown } from '@nestjs/common';

@Injectable()
export class DatabaseService implements OnModuleDestroy, BeforeApplicationShutdown, OnApplicationShutdown {
  constructor(private dataSource: DataSource) {}

  // Called when module is being destroyed
  async onModuleDestroy() {
    console.log('Module destroying...');
  }

  // Called before connections close
  async beforeApplicationShutdown(signal?: string) {
    console.log(`Received signal: ${signal}`);
    // Finish pending transactions
    await this.flushPendingWrites();
  }

  // Called after connections close
  async onApplicationShutdown(signal?: string) {
    console.log('Closing database connection...');
    await this.dataSource.destroy();
    console.log('Database connection closed');
  }
}
```

### Complete Shutdown Implementation

```typescript
@Injectable()
export class GracefulShutdownService implements OnApplicationShutdown {
  private isShuttingDown = false;
  private activeRequests = 0;

  constructor(
    private queueService: QueueService,
    private databaseService: DatabaseService,
    private cacheService: CacheService,
  ) {}

  incrementActiveRequests() {
    this.activeRequests++;
  }

  decrementActiveRequests() {
    this.activeRequests--;
  }

  isShutdown(): boolean {
    return this.isShuttingDown;
  }

  async onApplicationShutdown(signal?: string) {
    this.isShuttingDown = true;
    console.log(`Graceful shutdown initiated (${signal})`);

    // Wait for active requests to complete (with timeout)
    const timeout = 30000; // 30 seconds
    const start = Date.now();

    while (this.activeRequests > 0) {
      if (Date.now() - start > timeout) {
        console.warn(`Timeout waiting for ${this.activeRequests} requests`);
        break;
      }
      await new Promise(resolve => setTimeout(resolve, 100));
    }

    // Stop accepting new queue jobs
    await this.queueService.close();

    // Close cache connections
    await this.cacheService.close();

    // Close database last
    await this.databaseService.close();

    console.log('Graceful shutdown complete');
  }
}

// Middleware to track requests
@Injectable()
export class RequestTrackingMiddleware implements NestMiddleware {
  constructor(private shutdownService: GracefulShutdownService) {}

  use(req: Request, res: Response, next: NextFunction) {
    // Reject new requests during shutdown
    if (this.shutdownService.isShutdown()) {
      res.status(503).json({ message: 'Server is shutting down' });
      return;
    }

    this.shutdownService.incrementActiveRequests();
    res.on('finish', () => {
      this.shutdownService.decrementActiveRequests();
    });
    next();
  }
}
```

### Queue Graceful Shutdown

```typescript
@Injectable()
export class QueueService implements OnApplicationShutdown {
  constructor(@InjectQueue('tasks') private queue: Queue) {}

  async onApplicationShutdown() {
    console.log('Closing queue connections...');
    // Wait for active jobs to complete
    await this.queue.close();
    console.log('Queue closed');
  }
}
```

### Docker Stop Signal

```dockerfile
# Dockerfile
FROM node:20-alpine

WORKDIR /app
COPY . .
RUN npm ci --only=production
RUN npm run build

# Use exec form to receive signals
CMD ["node", "dist/main.js"]

# Allow 30 seconds for graceful shutdown
STOPSIGNAL SIGTERM
```

```yaml
# docker-compose.yml
services:
  app:
    build: .
    stop_grace_period: 30s
```

## Common Pitfalls

1. **Not enabling hooks**: Shutdown hooks are disabled by default.
2. **No timeout**: Waiting forever for hung requests.
3. **Wrong shutdown order**: Close external connections before database.

## Best Practices

- Enable shutdown hooks with `enableShutdownHooks()`
- Implement timeouts for waiting on requests
- Stop accepting requests during shutdown
- Close connections in correct order
- Log shutdown progress for debugging

## Summary

Graceful shutdown completes pending work before terminating. Enable shutdown hooks, implement cleanup in lifecycle methods, and set appropriate timeouts. Stop accepting new requests and close connections in the correct order.

## Resources

- [Lifecycle Events](https://docs.nestjs.com/fundamentals/lifecycle-events) — Shutdown hooks and lifecycle

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*