---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-clustering"
---

# Node.js Clustering

## Introduction

Node.js is single-threaded. Clustering spawns multiple worker processes to utilize all CPU cores. This multiplies throughput on multi-core servers without changing application code.

## Key Concepts

- **Cluster Module**: Node.js built-in multi-process support
- **Worker Process**: Child process handling requests
- **PM2**: Process manager with clustering built-in
- **IPC**: Inter-process communication

## Real World Context

Clustering maximizes:
- Multi-core CPU utilization
- Request throughput
- Application resilience (worker restart)
- Memory utilization across processes

## Deep Dive

### Native Clustering

```typescript
import * as cluster from 'cluster';
import * as os from 'os';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const numCPUs = os.cpus().length;

  if (cluster.isPrimary) {
    console.log(`Primary ${process.pid} is running`);

    // Fork workers
    for (let i = 0; i < numCPUs; i++) {
      cluster.fork();
    }

    cluster.on('exit', (worker, code, signal) => {
      console.log(`Worker ${worker.process.pid} died`);
      // Replace dead worker
      cluster.fork();
    });
  } else {
    // Workers run the NestJS app
    const app = await NestFactory.create(AppModule);
    await app.listen(3000);
    console.log(`Worker ${process.pid} started`);
  }
}

bootstrap();
```

### PM2 Configuration

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'nestjs-app',
    script: 'dist/main.js',
    instances: 'max',  // Use all CPUs
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
    },
    // Graceful shutdown
    kill_timeout: 5000,
    wait_ready: true,
    listen_timeout: 10000,
  }],
};
```

### Graceful Shutdown with Clustering

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Enable shutdown hooks
  app.enableShutdownHooks();

  await app.listen(3000);

  // Signal ready to PM2
  if (process.send) {
    process.send('ready');
  }

  // Graceful shutdown
  process.on('SIGINT', async () => {
    console.log('Received SIGINT, graceful shutdown...');
    await app.close();
    process.exit(0);
  });
}

bootstrap();
```

### Cluster-Aware Scheduling

```typescript
// When using @nestjs/schedule with clustering,
// only one instance should run scheduled jobs

// Option 1: Use a distributed lock
@Injectable()
export class ScheduledTasksService {
  constructor(private lockService: RedisLockService) {}

  @Cron('0 * * * *')
  async hourlyTask() {
    const lock = await this.lockService.acquireLock('hourly-task', 60000);
    if (!lock) {
      return; // Another worker has the lock
    }

    try {
      await this.runTask();
    } finally {
      await this.lockService.releaseLock('hourly-task');
    }
  }
}

// Option 2: Use queue for scheduled jobs
@Injectable()
export class SchedulerService {
  constructor(@InjectQueue('scheduled') private queue: Queue) {}

  @Cron('0 * * * *')
  async queueHourlyTask() {
    // Queue ensures single execution
    await this.queue.add('hourly', {}, {
      jobId: `hourly-${Date.now()}`,  // Prevent duplicates
    });
  }
}
```

### Load Testing Clustered App

```bash
# Start with PM2
pm2 start ecosystem.config.js

# Load test with autocannon
npx autocannon -c 100 -d 30 http://localhost:3000/api/health

# Monitor
pm2 monit
```

## Common Pitfalls

1. **Shared state between workers**: Workers don't share memory.
2. **Scheduled jobs running multiple times**: Use distributed locks.
3. **Uneven load distribution**: OS handles this automatically.

## Best Practices

- Use PM2 for production clustering
- Implement graceful shutdown
- Use distributed locks for scheduled jobs
- Monitor individual worker health
- Test with realistic load

## Summary

Clustering spawns multiple workers to utilize all CPU cores. Use PM2 for easy cluster management, implement graceful shutdown, and handle scheduled jobs with distributed locks. This multiplies throughput without code changes.

## Resources

- [Performance](https://docs.nestjs.com/techniques/performance) — Performance optimization

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*