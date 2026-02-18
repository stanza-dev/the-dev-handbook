---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-health-checks"
---

# Health Checks

## Introduction

Health checks report application status to orchestrators and load balancers. They determine if an instance should receive traffic, be restarted, or replaced.

## Key Concepts

- **Liveness**: Is the application running?
- **Readiness**: Can the application serve requests?
- **Health Indicators**: Individual dependency checks
- **Terminus**: NestJS health check module

## Real World Context

Health checks enable:
- Kubernetes pod management
- Load balancer traffic routing
- Automated recovery
- Monitoring dashboards

## Deep Dive

### Setup with Terminus

```bash
npm install @nestjs/terminus
```

```typescript
import { Module } from '@nestjs/common';
import { TerminusModule } from '@nestjs/terminus';
import { HttpModule } from '@nestjs/axios';
import { HealthController } from './health.controller';

@Module({
  imports: [TerminusModule, HttpModule],
  controllers: [HealthController],
})
export class HealthModule {}
```

### Comprehensive Health Controller

```typescript
import {
  Controller,
  Get,
} from '@nestjs/common';
import {
  HealthCheckService,
  HttpHealthIndicator,
  TypeOrmHealthIndicator,
  MemoryHealthIndicator,
  DiskHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private http: HttpHealthIndicator,
    private db: TypeOrmHealthIndicator,
    private memory: MemoryHealthIndicator,
    private disk: DiskHealthIndicator,
  ) {}

  @Get()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
      () => this.memory.checkHeap('memory_heap', 150 * 1024 * 1024), // 150MB
      () => this.memory.checkRSS('memory_rss', 300 * 1024 * 1024), // 300MB
    ]);
  }

  @Get('live')
  liveness() {
    return { status: 'ok' };
  }

  @Get('ready')
  readiness() {
    return this.health.check([
      () => this.db.pingCheck('database'),
      () => this.http.pingCheck('external-api', 'https://api.example.com/health'),
    ]);
  }
}
```

### Custom Health Indicator

```typescript
import { Injectable } from '@nestjs/common';
import { HealthIndicator, HealthIndicatorResult, HealthCheckError } from '@nestjs/terminus';

@Injectable()
export class RedisHealthIndicator extends HealthIndicator {
  constructor(private redis: Redis) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      await this.redis.ping();
      return this.getStatus(key, true);
    } catch (error) {
      throw new HealthCheckError(
        'Redis check failed',
        this.getStatus(key, false, { message: error.message }),
      );
    }
  }
}

// Usage
@Get()
check() {
  return this.health.check([
    () => this.redis.isHealthy('redis'),
  ]);
}
```

### Graceful Degradation in Health Checks

```typescript
@Get('ready')
async readiness() {
  const results = await Promise.allSettled([
    this.checkDatabase(),
    this.checkRedis(),
    this.checkExternalApi(),
  ]);

  const status = {
    database: results[0].status === 'fulfilled',
    redis: results[1].status === 'fulfilled',
    externalApi: results[2].status === 'fulfilled',
  };

  // App is ready if database is up (redis/api are optional)
  const isReady = status.database;

  return {
    status: isReady ? 'ok' : 'error',
    details: status,
  };
}
```

## Common Pitfalls

1. **Heavy health checks**: Keep checks fast (< 1s).
2. **Cascading failures**: External API down shouldn't fail liveness.
3. **No timeouts**: Hung checks block health endpoint.

## Best Practices

- Separate liveness (is running) from readiness (can serve)
- Keep checks lightweight and fast
- Use timeouts for all checks
- Only fail readiness for critical dependencies
- Log health check failures

## Summary

Health checks report application status to orchestrators. Use liveness for basic aliveness, readiness for serving capability. Create custom indicators for specific dependencies and implement graceful degradation.

## Resources

- [Health Checks (Terminus)](https://docs.nestjs.com/recipes/terminus) — Official health checks guide

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*