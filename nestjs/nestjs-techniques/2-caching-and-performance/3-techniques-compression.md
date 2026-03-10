---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-compression"
---

# Compression

## Introduction

Response compression reduces bandwidth usage and improves load times. Large JSON responses and assets can be compressed on the fly, dramatically reducing transfer sizes for clients.

## Key Concepts

- **gzip/deflate**: Common compression algorithms
- **compression middleware**: Express middleware for response compression
- **Threshold**: Minimum size before compression kicks in

## Real World Context

HTTP responses can be significantly reduced in size with compression. A 500KB JSON API response compresses to roughly 50KB with gzip, reducing bandwidth costs and improving response times for mobile users on slow connections.

## Deep Dive

### Express Setup

Install the `compression` middleware and its type definitions.

```bash
npm install compression
npm install -D @types/compression
```

Apply the middleware in `bootstrap()` to compress all outgoing responses.

```typescript
import * as compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(compression());
  await app.listen(3000);
}
```

With zero configuration, this uses gzip and only compresses responses that the client accepts.

### Custom Configuration

Pass an options object to fine-tune compression behavior, including a size threshold and custom filter.

```typescript
app.use(compression({
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  },
  threshold: 1024, // Only compress responses > 1KB
  level: 6, // Compression level (1-9)
}));
```

The `level` option trades CPU for compression ratio; level 6 is a good default balance.

### Fastify Compression

When using Fastify instead of Express, register `@fastify/compress` via the application instance.

```typescript
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';
import fastifyCompress from '@fastify/compress';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter(),
  );
  await app.register(fastifyCompress);
  await app.listen(3000, '0.0.0.0');
}
```

Note the `'0.0.0.0'` binding, which is required for Fastify to accept connections from outside the container.

## Common Pitfalls

1. **Double compression**: If behind a reverse proxy that compresses, don't compress twice.
2. **CPU overhead**: Compression uses CPU. Balance with caching.
3. **Small responses**: Don't compress tiny responses—overhead isn't worth it.

## Best Practices

- Set a reasonable threshold (1KB+)
- Consider pre-compressing static assets
- Monitor CPU usage in high-traffic scenarios
- Use CDN for static asset compression

## Summary

- Compression middleware reduces response sizes using gzip/deflate algorithms
- Set a threshold (e.g., 1KB) to avoid compressing tiny responses
- Choose the compression library based on platform (Express vs Fastify)
- Avoid double compression when behind a reverse proxy that already compresses

## Code Examples

**Enabling response compression middleware in a NestJS application**

```typescript
import * as compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(compression());
  await app.listen(3000);
}
```


## Resources

- [Compression](https://docs.nestjs.com/techniques/compression) — Official compression guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*