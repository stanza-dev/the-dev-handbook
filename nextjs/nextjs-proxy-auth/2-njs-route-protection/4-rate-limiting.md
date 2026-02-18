---
source_course: "nextjs-proxy-auth"
source_lesson: "nextjs-proxy-auth-njs-rate-limiting"
---

# Rate Limiting in Proxy

## Introduction

Without rate limiting, a single user—or bot—can hammer your server with thousands of requests per second. Proxy is the perfect place to implement rate limiting because it catches abuse before your route handlers even execute.

## Key Concepts

**Rate limiting** restricts how many requests a client can make in a given time window. Common strategies:

- **Fixed window**: 100 requests per minute, resets at minute boundaries
- **Sliding window**: 100 requests in any 60-second period
- **Token bucket**: Refills tokens over time, burst-friendly

## Real World Context

Rate limiting protects against:
- Brute force attacks on login endpoints
- API abuse from aggressive scrapers
- Accidental denial of service from buggy clients
- Cost overruns from excessive API calls

## Deep Dive

### Simple In-Memory Rate Limiter

For single-instance deployments:

```typescript
import { NextRequest, NextResponse } from 'next/server';

// Simple in-memory store (use Redis in production)
const rateLimitMap = new Map<string, { count: number; timestamp: number }>();

const RATE_LIMIT = 100; // requests
const WINDOW_MS = 60 * 1000; // 1 minute

function getRateLimitKey(request: NextRequest): string {
  // Use IP address as identifier
  const ip = request.ip || request.headers.get('x-forwarded-for') || 'unknown';
  return ip;
}

export function proxy(request: NextRequest) {
  const key = getRateLimitKey(request);
  const now = Date.now();
  
  const record = rateLimitMap.get(key);
  
  if (!record || now - record.timestamp > WINDOW_MS) {
    // New window
    rateLimitMap.set(key, { count: 1, timestamp: now });
  } else if (record.count >= RATE_LIMIT) {
    // Rate limit exceeded
    return NextResponse.json(
      { error: 'Too many requests. Please try again later.' },
      { 
        status: 429,
        headers: {
          'Retry-After': String(Math.ceil((record.timestamp + WINDOW_MS - now) / 1000)),
        },
      }
    );
  } else {
    // Increment counter
    record.count++;
  }

  return NextResponse.next();
}
```

### Per-Endpoint Rate Limits

Different endpoints need different limits:

```typescript
const LIMITS = {
  '/api/login': { limit: 5, windowMs: 60000 },      // 5 per minute
  '/api/register': { limit: 3, windowMs: 60000 },   // 3 per minute
  '/api/search': { limit: 30, windowMs: 60000 },    // 30 per minute
  default: { limit: 100, windowMs: 60000 },         // 100 per minute
};

function getLimitConfig(pathname: string) {
  for (const [path, config] of Object.entries(LIMITS)) {
    if (path !== 'default' && pathname.startsWith(path)) {
      return config;
    }
  }
  return LIMITS.default;
}
```

### Redis-Based Rate Limiting (Production)

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

export async function proxy(request: NextRequest) {
  const ip = request.ip || 'unknown';
  const key = `ratelimit:${ip}`;
  
  const [count] = await redis
    .multi()
    .incr(key)
    .expire(key, 60) // 60 second window
    .exec();

  if ((count as number) > 100) {
    return NextResponse.json(
      { error: 'Rate limit exceeded' },
      { status: 429 }
    );
  }

  return NextResponse.next();
}
```

### Adding Rate Limit Headers

```typescript
const response = NextResponse.next();
response.headers.set('X-RateLimit-Limit', String(RATE_LIMIT));
response.headers.set('X-RateLimit-Remaining', String(RATE_LIMIT - count));
response.headers.set('X-RateLimit-Reset', String(resetTimestamp));
return response;
```

## Common Pitfalls

1. **In-memory limits don't scale**: With multiple server instances, each has its own counter. Use Redis for distributed deployments.

2. **IP-based limits block shared IPs**: Corporate networks and VPNs share IPs. Consider user-based limits for authenticated routes.

3. **Forgetting the Retry-After header**: Clients need to know when they can retry. Include this header in 429 responses.

## Best Practices

- **Use Redis for production**: In-memory stores don't work with multiple instances
- **Different limits for different endpoints**: Login needs stricter limits than read endpoints
- **Include rate limit headers**: Help good-faith clients stay within limits
- **Monitor and alert**: Track when users hit rate limits—it could indicate attacks or bugs

## Summary

Rate limiting in Proxy protects your application at the network edge. Use IP-based identification, implement per-endpoint limits, and always include proper headers. For production, use Redis or a dedicated rate limiting service like Upstash.

## Resources

- [Upstash Rate Limiting](https://upstash.com/docs/oss/sdks/ts/ratelimit/overview) — Production rate limiting with Upstash

---

> 📘 *This lesson is part of the [Next.js Proxy & Authentication](https://stanza.dev/courses/nextjs-proxy-auth) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*