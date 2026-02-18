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

```bash
npm install @nestjs/throttler
```

```typescript
import { ThrottlerModule, ThrottlerGuard } from '@nestjs/throttler';
import { APP_GUARD } from '@nestjs/core';

@Module({
  imports: [
    ThrottlerModule.forRoot([{
      ttl: 60000,    // 60 seconds
      limit: 10,     // 10 requests per TTL
    }]),
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

### Multiple Rate Limit Tiers

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

### Skip Rate Limiting

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

### Custom Throttle per Route

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

### Custom Storage (Redis)

For distributed applications:

```typescript
import { ThrottlerStorageRedisService } from '@nestjs/throttler-storage-redis';

ThrottlerModule.forRoot({
  throttlers: [{ limit: 10, ttl: 60000 }],
  storage: new ThrottlerStorageRedisService(redisClient),
})
```

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

The @nestjs/throttler module provides rate limiting with configurable TTL and limit values. Apply globally with ThrottlerGuard, customize per-route with @Throttle(), or skip with @SkipThrottle(). Use distributed storage like Redis for production deployments.

## Resources

- [Rate Limiting Documentation](https://docs.nestjs.com/security/rate-limiting) — Official rate limiting guide

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*