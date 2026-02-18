---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-graceful-degradation"
---

# Graceful Degradation

## Introduction

When dependencies fail, your application shouldn't crash entirely. Graceful degradation provides reduced functionality or fallback responses instead of complete failure. Users get a degraded experience rather than error pages.

## Key Concepts

- **Fallback**: Alternative response when primary fails
- **Feature Flags**: Toggle features on/off at runtime
- **Bulkhead**: Isolate failures to prevent cascade
- **Timeout**: Don't wait forever for slow dependencies

## Real World Context

Graceful degradation examples:
- Show cached data when API is down
- Disable recommendations but keep core features
- Return partial data instead of failing completely
- Queue requests for later when service is unavailable

## Deep Dive

### Fallback Pattern

```typescript
@Injectable()
export class ProductService {
  constructor(
    private recommendationService: RecommendationService,
    private cacheService: CacheService,
  ) {}

  async getProductWithRecommendations(productId: string) {
    const product = await this.productsRepo.findById(productId);
    if (!product) throw new NotFoundException();

    // Try to get recommendations with fallback
    let recommendations: Product[];
    try {
      recommendations = await this.recommendationService.getForProduct(productId);
    } catch (error) {
      // Fallback to cached or empty recommendations
      recommendations = await this.getFallbackRecommendations(productId);
    }

    return { product, recommendations };
  }

  private async getFallbackRecommendations(productId: string): Promise<Product[]> {
    // Try cache
    const cached = await this.cacheService.get(`recs:${productId}`);
    if (cached) return cached;

    // Return popular products as fallback
    return this.productsRepo.findPopular(4);
  }
}
```

### Timeout Wrapper

```typescript
export async function withTimeout<T>(
  promise: Promise<T>,
  ms: number,
  fallback?: T,
): Promise<T> {
  const timeout = new Promise<T>((_, reject) =>
    setTimeout(
      () => reject(new Error(`Operation timed out after ${ms}ms`)),
      ms,
    ),
  );

  try {
    return await Promise.race([promise, timeout]);
  } catch (error) {
    if (fallback !== undefined) {
      return fallback;
    }
    throw error;
  }
}

// Usage
async getRecommendations(userId: string) {
  return withTimeout(
    this.mlService.getRecommendations(userId),
    2000, // 2 second timeout
    [], // Return empty array as fallback
  );
}
```

### Feature Flags for Degradation

```typescript
@Injectable()
export class FeatureFlagService {
  private flags = new Map<string, boolean>();

  async isEnabled(flag: string): Promise<boolean> {
    // Could be from database, config service, or external service
    return this.flags.get(flag) ?? true;
  }

  async setFlag(flag: string, enabled: boolean): Promise<void> {
    this.flags.set(flag, enabled);
  }
}

@Injectable()
export class CheckoutService {
  async checkout(order: Order) {
    // Core checkout always runs
    const result = await this.processPayment(order);

    // Optional features with flags
    if (await this.featureFlags.isEnabled('send-confirmation-email')) {
      try {
        await this.emailService.sendConfirmation(order);
      } catch (error) {
        // Log but don't fail checkout
        this.logger.error('Failed to send confirmation email', error);
      }
    }

    if (await this.featureFlags.isEnabled('update-inventory')) {
      try {
        await this.inventoryService.decrementStock(order.items);
      } catch (error) {
        // Queue for later processing
        await this.queue.add('sync-inventory', order);
      }
    }

    return result;
  }
}
```

### Bulkhead Pattern

```typescript
@Injectable()
export class ApiService {
  private semaphores = new Map<string, number>();
  private maxConcurrent = 10;

  async callWithBulkhead<T>(
    service: string,
    fn: () => Promise<T>,
  ): Promise<T> {
    const current = this.semaphores.get(service) ?? 0;
    
    if (current >= this.maxConcurrent) {
      throw new Error(`Bulkhead limit reached for ${service}`);
    }

    this.semaphores.set(service, current + 1);
    
    try {
      return await fn();
    } finally {
      this.semaphores.set(service, (this.semaphores.get(service) ?? 1) - 1);
    }
  }
}
```

### Health Check with Degradation Status

```typescript
@Controller('health')
export class HealthController {
  @Get()
  async check() {
    const services = await Promise.allSettled([
      this.checkDatabase(),
      this.checkRedis(),
      this.checkExternalApi(),
    ]);

    const results = {
      database: services[0].status === 'fulfilled',
      redis: services[1].status === 'fulfilled',
      externalApi: services[2].status === 'fulfilled',
    };

    const allHealthy = Object.values(results).every(Boolean);
    const status = allHealthy ? 'healthy' : 'degraded';

    return {
      status,
      services: results,
      timestamp: new Date().toISOString(),
    };
  }
}
```

## Common Pitfalls

1. **Silent failures**: Log degraded state for monitoring.
2. **Fallback causing more load**: Cache fallbacks to avoid cascading.
3. **No monitoring**: Can't fix what you can't see.

## Best Practices

- Define fallbacks for non-critical features
- Use timeouts for all external calls
- Implement feature flags for quick degradation
- Monitor and alert on degraded state
- Test failure scenarios regularly

## Summary

Graceful degradation keeps core functionality working when dependencies fail. Use fallbacks, timeouts, feature flags, and bulkheads. Monitor degraded state and test failure scenarios. Users prefer limited functionality over error pages.

## Resources

- [Health Checks](https://docs.nestjs.com/recipes/terminus) — Health checks and monitoring

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*