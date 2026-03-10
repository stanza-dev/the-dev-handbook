---
source_course: "nestjs-security"
source_lesson: "nestjs-security-rate-limiting"
---

# Rate Limiting

## Introduction

APIs are vulnerable to abuse—whether malicious attacks or accidental overuse. Rate limiting controls how many requests a client can make in a given time window, protecting your server from being overwhelmed and ensuring fair resource distribution.

## Key Concepts

- **Rate Limiting**: Restricting the number of requests per time window
- **Throttling**: Another term for rate limiting
- **TTL (Time To Live)**: The duration of the rate limit window
- **Limit**: Maximum requests allowed within the TTL

## Real World Context

Rate limiting protects against:
- **Brute-force attacks**: Password guessing, API key enumeration
- **DDoS**: Distributed denial-of-service attacks
- **Scraping**: Automated data extraction
- **Cost overruns**: Limiting expensive operations

## Deep Dive

### Installation & Basic Setup

Install the official NestJS throttler package.

```bash
npm install @nestjs/throttler
```

Configure the module with a throttler definition and register the guard globally using `APP_GUARD`.

```typescript
import { ThrottlerModule, ThrottlerGuard } from '@nestjs/throttler';
import { APP_GUARD } from '@nestjs/core';

@Module({
  imports: [
    ThrottlerModule.forRoot({
      throttlers: [{
        ttl: 60000,    // 60 seconds
        limit: 10,     // 10 requests per TTL
      }],
    }),
  ],
  providers: [
    {
      provide: APP_GUARD,
      useClass: ThrottlerGuard,
    },
  ],
})
export class AppModule {}
```

Registering `ThrottlerGuard` as `APP_GUARD` applies rate limiting to every route automatically — no need to add `@UseGuards()` individually.

### Multiple Rate Limit Tiers

Define multiple named throttlers to enforce different rate limits simultaneously — a request must pass all tiers to proceed.

```typescript
ThrottlerModule.forRoot([
  {
    name: 'short',
    ttl: 1000,    // 1 second
    limit: 3,     // 3 requests per second
  },
  {
    name: 'medium',
    ttl: 10000,   // 10 seconds
    limit: 20,    // 20 requests per 10 seconds
  },
  {
    name: 'long',
    ttl: 60000,   // 1 minute
    limit: 100,   // 100 requests per minute
  },
])
```

This layered approach catches both burst abuse (short tier) and sustained abuse (long tier), providing more nuanced protection.

### Skip Rate Limiting

Exempt specific routes or entire controllers from rate limiting using the `@SkipThrottle()` decorator — useful for health checks and public endpoints.

```typescript
import { SkipThrottle } from '@nestjs/throttler';

@Controller('health')
export class HealthController {
  @SkipThrottle()
  @Get()
  check() {
    return { status: 'ok' };
  }
}

// Or skip entire controller
@SkipThrottle()
@Controller('public')
export class PublicController {}
```

Applying `@SkipThrottle()` at the controller level exempts all routes within that controller.

### Custom Throttle per Route

Override the global rate limit on sensitive routes using the `@Throttle()` decorator to apply stricter limits.

```typescript
import { Throttle } from '@nestjs/throttler';

@Controller('auth')
export class AuthController {
  @Throttle({ default: { limit: 3, ttl: 60000 } })
  @Post('login')
  login() {
    // Only 3 login attempts per minute
  }
}
```

Login endpoints are prime targets for brute-force attacks, so applying a much tighter limit (3 per minute vs. the global 10) is a critical hardening step.

### Custom Storage (Redis)

For distributed applications running multiple server instances, use Redis-backed storage so rate limits are shared across all nodes.

```typescript
import { ThrottlerStorageRedisService } from '@nestjs/throttler-storage-redis';
import Redis from 'ioredis';

ThrottlerModule.forRoot({
  throttlers: [
    { name: 'short', ttl: 1000, limit: 3 },
    { name: 'long', ttl: 60000, limit: 100 },
  ],
  storage: new ThrottlerStorageRedisService(new Redis()),
})
```

Without Redis, each server instance tracks limits independently — an attacker could multiply their allowed requests by the number of instances behind the load balancer.

## Common Pitfalls

1. **Not using distributed storage**: In-memory storage doesn't work across multiple server instances. Use Redis for production.
2. **Rate limiting by IP only**: Consider user ID or API key for authenticated endpoints.
3. **Too restrictive limits**: Start generous and tighten based on actual usage data.

## Best Practices

- Use Redis or similar for distributed rate limiting
- Implement different limits for different endpoint types
- Return `Retry-After` header when rate limited
- Log rate limit hits for security monitoring
- Consider user-based limits for authenticated routes

## Summary

- Use `@nestjs/throttler` to configure rate limiting with TTL (time window) and limit (max requests)
- Apply globally via `APP_GUARD` with `ThrottlerGuard`, or customize per-route with `@Throttle()`
- Skip rate limiting on specific routes or controllers using `@SkipThrottle()`
- Use Redis-backed storage for distributed/multi-instance production deployments

## Code Examples

**Configuring ThrottlerModule with rate limit settings and applying ThrottlerGuard globally**

```typescript
import { ThrottlerModule, ThrottlerGuard } from '@nestjs/throttler';
import { APP_GUARD } from '@nestjs/core';

@Module({
  imports: [
    ThrottlerModule.forRoot({
      throttlers: [{
        ttl: 60000,    // 60 seconds
        limit: 10,     // 10 requests per TTL
      }],
    }),
  ],
  providers: [
    {
      provide: APP_GUARD,
      useClass: ThrottlerGuard,
    },
  ],
})
export class AppModule {}
```


## Resources

- [Rate Limiting Documentation](https://docs.nestjs.com/security/rate-limiting) — Official rate limiting guide

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*