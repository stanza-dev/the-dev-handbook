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

## Deep Dive

### Express Setup

```bash
npm install compression
npm install -D @types/compression
```

```typescript
import * as compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(compression());
  await app.listen(3000);
}
```

### Custom Configuration

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

### Fastify Compression

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

Compression middleware reduces response sizes using gzip/deflate. Configure threshold to avoid compressing tiny responses. Consider platform (Express vs Fastify) when choosing compression libraries.

## Resources

- [Compression](https://docs.nestjs.com/techniques/compression) — Official compression guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*