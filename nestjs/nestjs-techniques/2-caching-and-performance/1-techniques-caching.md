---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-caching"
---

# Caching

## Introduction

Database queries, API calls, and complex computations take time. Caching stores results for fast retrieval, dramatically improving response times. NestJS provides a unified caching API that works with in-memory stores or Redis.

## Key Concepts

- **CacheModule**: Module for configuring cache stores
- **CacheInterceptor**: Auto-caches GET responses
- **@CacheKey()**: Custom cache key
- **@CacheTTL()**: Custom time-to-live

## Real World Context

Database queries and external API calls are the main performance bottlenecks in web applications. A product listing page hitting the database on every request wastes resources when the data changes once per hour. Caching stores frequently accessed data in memory for instant retrieval.

## Deep Dive

### Basic Setup

Import and register `CacheModule` to enable in-memory caching with default settings.

```typescript
import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [CacheModule.register()],
})
export class AppModule {}
```

By default this uses an in-memory store, which is suitable for development and single-instance deployments.

### Auto-Caching with Interceptor

Apply `CacheInterceptor` at the controller level to automatically cache all GET responses.

```typescript
@Controller('products')
@UseInterceptors(CacheInterceptor)
export class ProductsController {
  @Get()
  findAll() {
    // This response is cached automatically
    return this.productsService.findAll();
  }

  @Get(':id')
  @CacheTTL(30000) // 30 seconds
  findOne(@Param('id') id: string) {
    return this.productsService.findOne(id);
  }
}
```

Use `@CacheTTL()` to override the default time-to-live on specific endpoints.

### Manual Cache Access

Inject `CACHE_MANAGER` directly for fine-grained control over cache reads, writes, and invalidation.

```typescript
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import { Cache } from 'cache-manager';

@Injectable()
export class ProductsService {
  constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}

  async findOne(id: string): Promise<Product> {
    const cacheKey = \`product:\${id}\`;
    const cached = await this.cacheManager.get<Product>(cacheKey);
    
    if (cached) return cached;
    
    const product = await this.productRepository.findOne(id);
    await this.cacheManager.set(cacheKey, product, 60000);
    return product;
  }

  async update(id: string, dto: UpdateProductDto): Promise<Product> {
    const product = await this.productRepository.update(id, dto);
    await this.cacheManager.del(\`product:\${id}\`);
    return product;
  }
}
```

Notice how `update()` calls `del()` to invalidate the cached entry, preventing stale data.

### Redis Store (Keyv)

Since `cache-manager` v5, stores use Keyv adapters. Install the Redis adapter:

```bash
npm install @keyv/redis
```

Register the Redis store with `registerAsync()` and pass a connection URL.

```typescript
import KeyvRedis from '@keyv/redis';

CacheModule.registerAsync({
  useFactory: async () => ({
    stores: [
      new KeyvRedis('redis://localhost:6379'),
    ],
  }),
})
```

You can also combine multiple stores (in-memory primary + Redis fallback):

```typescript
import KeyvRedis from '@keyv/redis';
import { Keyv } from 'keyv';
import { CacheableMemory } from 'cacheable';

CacheModule.registerAsync({
  useFactory: async () => ({
    stores: [
      new Keyv({ store: new CacheableMemory({ ttl: 60000, lruSize: 5000 }) }),
      new KeyvRedis('redis://localhost:6379'),
    ],
  }),
})
```

With a tiered setup, reads hit the fast in-memory store first and fall back to Redis on a miss.

## Common Pitfalls

1. **Stale data**: Cache invalidation is hard. Invalidate on updates.
2. **Cache stampede**: Many requests for uncached data simultaneously. Use lock patterns.
3. **Caching user-specific data**: CacheInterceptor caches by URL—doesn't include user context.

## Best Practices

- Use Redis for production (supports clustering, persistence)
- Invalidate cache on data changes
- Set appropriate TTLs based on data volatility
- Use custom cache keys for user-specific data

## Summary

NestJS caching uses CacheModule with in-memory or Redis stores. Use CacheInterceptor for automatic response caching, or inject CACHE_MANAGER for manual control. Always invalidate cache when data changes.

## Code Examples

**Registering the CacheModule with default in-memory cache settings**

```typescript
import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [CacheModule.register()],
})
export class AppModule {}
```


## Resources

- [Caching](https://docs.nestjs.com/techniques/caching) — Official caching guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*