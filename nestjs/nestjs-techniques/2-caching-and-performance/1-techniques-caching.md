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

## Deep Dive

### Basic Setup

```typescript
import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [CacheModule.register()],
})
export class AppModule {}
```

### Auto-Caching with Interceptor

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

### Manual Cache Access

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

### Redis Store

```typescript
import { redisStore } from 'cache-manager-redis-yet';

CacheModule.registerAsync({
  useFactory: async () => ({
    store: await redisStore({
      socket: { host: 'localhost', port: 6379 },
      ttl: 60000,
    }),
  }),
})
```

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

## Resources

- [Caching](https://docs.nestjs.com/techniques/caching) — Official caching guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*