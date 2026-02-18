---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-caching-strategies"
---

# Caching Strategies

## Introduction

Caching stores computed results for fast retrieval. The right caching strategy dramatically reduces database load and response times. Understanding cache invalidation, TTLs, and cache levels is essential for performance.

## Key Concepts

- **Cache Aside**: Application manages cache explicitly
- **Write Through**: Write to cache and database simultaneously
- **Cache Invalidation**: Removing stale cached data
- **TTL (Time To Live)**: Automatic cache expiration

## Real World Context

Caching reduces:
- Database query load
- API response times
- External service calls
- Computed result recalculation

## Deep Dive

### Cache Aside Pattern

```typescript
@Injectable()
export class ProductsService {
  constructor(
    @Inject(CACHE_MANAGER) private cacheManager: Cache,
    private productsRepo: Repository<Product>,
  ) {}

  async findById(id: string): Promise<Product> {
    const cacheKey = `product:${id}`;

    // Check cache first
    const cached = await this.cacheManager.get<Product>(cacheKey);
    if (cached) return cached;

    // Cache miss: fetch from database
    const product = await this.productsRepo.findOne({ where: { id } });
    if (!product) throw new NotFoundException();

    // Store in cache
    await this.cacheManager.set(cacheKey, product, 3600000); // 1 hour

    return product;
  }

  async update(id: string, dto: UpdateProductDto): Promise<Product> {
    await this.productsRepo.update(id, dto);
    const product = await this.productsRepo.findOne({ where: { id } });

    // Invalidate cache
    await this.cacheManager.del(`product:${id}`);

    return product;
  }
}
```

### Multi-Level Caching

```typescript
@Injectable()
export class MultiLevelCacheService {
  private localCache = new Map<string, { value: any; expiry: number }>();

  constructor(@Inject(CACHE_MANAGER) private redis: Cache) {}

  async get<T>(key: string): Promise<T | null> {
    // L1: Local memory cache
    const local = this.localCache.get(key);
    if (local && local.expiry > Date.now()) {
      return local.value;
    }

    // L2: Redis cache
    const remote = await this.redis.get<T>(key);
    if (remote) {
      // Populate L1 cache
      this.localCache.set(key, {
        value: remote,
        expiry: Date.now() + 60000, // 1 minute local cache
      });
      return remote;
    }

    return null;
  }

  async set(key: string, value: any, ttl: number) {
    await this.redis.set(key, value, ttl);
    this.localCache.set(key, {
      value,
      expiry: Date.now() + Math.min(ttl, 60000),
    });
  }
}
```

### Cache Warming

```typescript
@Injectable()
export class CacheWarmingService implements OnApplicationBootstrap {
  async onApplicationBootstrap() {
    await this.warmCache();
  }

  @Cron('0 */30 * * * *')  // Every 30 minutes
  async warmCache() {
    // Pre-populate frequently accessed data
    const popularProducts = await this.productsRepo.find({
      where: { featured: true },
    });

    for (const product of popularProducts) {
      await this.cacheManager.set(
        `product:${product.id}`,
        product,
        3600000,
      );
    }

    this.logger.log(`Warmed cache with ${popularProducts.length} products`);
  }
}
```

### Cache Tags for Invalidation

```typescript
@Injectable()
export class TaggedCacheService {
  private tags = new Map<string, Set<string>>();

  async setWithTags(key: string, value: any, tags: string[], ttl: number) {
    await this.cacheManager.set(key, value, ttl);

    for (const tag of tags) {
      if (!this.tags.has(tag)) {
        this.tags.set(tag, new Set());
      }
      this.tags.get(tag).add(key);
    }
  }

  async invalidateTag(tag: string) {
    const keys = this.tags.get(tag);
    if (!keys) return;

    for (const key of keys) {
      await this.cacheManager.del(key);
    }
    this.tags.delete(tag);
  }
}

// Usage
await this.cache.setWithTags(
  `product:${id}`,
  product,
  ['products', `category:${product.categoryId}`],
  3600000,
);

// Invalidate all products in a category
await this.cache.invalidateTag(`category:${categoryId}`);
```

### HTTP Response Caching

```typescript
@Controller('products')
@UseInterceptors(CacheInterceptor)
export class ProductsController {
  @Get()
  @CacheTTL(300000)  // 5 minutes
  findAll() {
    return this.productsService.findAll();
  }

  @Get(':id')
  @CacheKey('product')  // Custom cache key
  @CacheTTL(600000)  // 10 minutes
  findOne(@Param('id') id: string) {
    return this.productsService.findById(id);
  }
}
```

## Common Pitfalls

1. **Stale data**: Cache invalidation on updates is crucial.
2. **Cache stampede**: Many requests miss cache simultaneously.
3. **Over-caching**: Caching volatile data wastes memory.

## Best Practices

- Cache expensive computations and frequent queries
- Use appropriate TTLs based on data volatility
- Implement cache invalidation on data changes
- Use multi-level caching for high-traffic data
- Monitor cache hit rates

## Summary

Caching strategies include cache-aside, write-through, and multi-level approaches. Use TTLs for automatic expiration, invalidation on updates, and cache warming for critical data. Monitor hit rates to tune effectiveness.

## Resources

- [Caching](https://docs.nestjs.com/techniques/caching) — Caching techniques

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*