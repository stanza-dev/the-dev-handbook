---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-compression"
---

# Compression

## Introduction

Response compression reduces payload sizes by 70-90%, dramatically improving load times for clients on slow connections. The CPU cost is usually worth the bandwidth savings.

## Key Concepts

- **gzip**: Most widely supported compression algorithm
- **Brotli**: Better compression ratio, growing browser support
- **Threshold**: Minimum response size before compression
- **Content-Type filtering**: Only compress text-based responses

## Real World Context

A typical JSON API response might be 50KB uncompressed. With gzip at level 6, it drops to around 8KB — an 84% reduction. For mobile users on 3G connections, this means pages load in seconds instead of timing out.

## Deep Dive

### Express Compression

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

### Configuration Options

```typescript
app.use(compression({
  threshold: 1024, // Only compress responses > 1KB
  level: 6, // Compression level 1-9 (higher = more CPU)
  filter: (req, res) => {
    // Don't compress if x-no-compression header
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  },
}));
```

### Fastify Compression

```typescript
import compression from '@fastify/compress';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter(),
  );
  
  await app.register(compression, {
    encodings: ['gzip', 'deflate'],
    threshold: 1024,
  });
  
  await app.listen(3000, '0.0.0.0');
}
```

### Brotli Compression

By default, `@fastify/compress` uses **Brotli** compression on Node >= 11.7.0 when browsers indicate support. Brotli achieves better compression ratios than gzip but is more CPU-intensive. You can tune the quality parameter (0-11, default 11):

```typescript
import { constants } from 'node:zlib';

await app.register(compression, {
  brotliOptions: {
    params: { [constants.BROTLI_PARAM_QUALITY]: 4 }, // Faster compression
  },
});
```

Set a lower quality (e.g. 4) for a good balance between compression ratio and CPU usage. Quality 11 produces the smallest output but is significantly slower.

### When NOT to Compress

- Already compressed content (images, videos, zipped files)
- Very small responses (< 1KB—overhead isn't worth it)
- When behind a CDN/proxy that handles compression

## Common Pitfalls

1. **Double compression**: If CDN/proxy compresses, don't compress again.
2. **Compressing images**: Images are already compressed. Skip them.
3. **High compression level**: Level 9 uses significantly more CPU for marginal gains.

## Best Practices

- Set threshold to avoid compressing tiny responses
- Use level 6 (good balance of speed/compression)
- Monitor CPU usage in high-traffic scenarios
- Let CDN handle compression for static assets

## Summary

Compression reduces response sizes significantly. Use compression middleware for Express or @fastify/compress for Fastify. Brotli is the default for Fastify on modern Node.js and offers better ratios than gzip — tune its quality level for your workload. Set appropriate thresholds and avoid compressing already-compressed content.

## Code Examples

**Registering @fastify/compress with Brotli quality tuning — lower quality (e.g. 4) trades compression ratio for speed**

```typescript
import compression from '@fastify/compress';
import { constants } from 'node:zlib';

// Inside bootstrap()
const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  new FastifyAdapter(),
);

await app.register(compression, {
  brotliOptions: {
    params: { [constants.BROTLI_PARAM_QUALITY]: 4 },
  },
});
```


## Resources

- [Compression](https://docs.nestjs.com/techniques/compression) — Official compression guide

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*