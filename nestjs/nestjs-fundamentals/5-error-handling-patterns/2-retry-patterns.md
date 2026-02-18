---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-retry-patterns"
---

# Retry Patterns

## Introduction

External services fail temporarily—network timeouts, rate limits, service restarts. Retry patterns add resilience by automatically retrying failed operations with appropriate delays and limits.

## Key Concepts

- **Exponential Backoff**: Increasing delay between retries
- **Jitter**: Random variation to prevent thundering herd
- **Circuit Breaker**: Stop retrying when service is definitely down
- **Idempotency**: Safe to retry without duplicate effects

## Real World Context

Retry patterns handle:
- Temporary network failures
- Rate-limited API calls
- Database connection drops
- Service restarts

## Deep Dive

### Simple Retry Utility

```typescript
export async function retry<T>(
  fn: () => Promise<T>,
  options: {
    maxAttempts: number;
    baseDelay: number;
    maxDelay?: number;
    shouldRetry?: (error: Error) => boolean;
  },
): Promise<T> {
  const { maxAttempts, baseDelay, maxDelay = 30000, shouldRetry = () => true } = options;
  
  let lastError: Error;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      
      if (attempt === maxAttempts || !shouldRetry(lastError)) {
        throw lastError;
      }
      
      // Exponential backoff with jitter
      const delay = Math.min(
        baseDelay * Math.pow(2, attempt - 1) + Math.random() * 1000,
        maxDelay,
      );
      
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  
  throw lastError!;
}
```

### Retry Decorator

```typescript
export function Retry(options: RetryOptions) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor,
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      return retry(
        () => originalMethod.apply(this, args),
        options,
      );
    };

    return descriptor;
  };
}

// Usage
@Injectable()
export class PaymentService {
  @Retry({ maxAttempts: 3, baseDelay: 1000 })
  async processPayment(orderId: string): Promise<PaymentResult> {
    return this.paymentGateway.charge(orderId);
  }
}
```

### Circuit Breaker Pattern

```typescript
export class CircuitBreaker {
  private failures = 0;
  private lastFailureTime: number = 0;
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';

  constructor(
    private readonly threshold: number = 5,
    private readonly resetTimeout: number = 30000,
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.resetTimeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }

  private onFailure() {
    this.failures++;
    this.lastFailureTime = Date.now();
    
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
    }
  }
}
```

### Service with Circuit Breaker

```typescript
@Injectable()
export class ExternalApiService {
  private circuitBreaker = new CircuitBreaker(5, 30000);

  async fetchData(): Promise<Data> {
    return this.circuitBreaker.execute(async () => {
      return retry(
        () => this.httpService.get('/api/data'),
        { maxAttempts: 3, baseDelay: 1000 },
      );
    });
  }
}
```

### Idempotency for Safe Retries

```typescript
@Injectable()
export class PaymentService {
  async processPayment(
    orderId: string,
    idempotencyKey: string,
  ): Promise<PaymentResult> {
    // Check if already processed
    const existing = await this.paymentsRepo.findByIdempotencyKey(idempotencyKey);
    if (existing) {
      return existing;
    }

    // Process payment
    const result = await this.paymentGateway.charge(orderId);

    // Store with idempotency key
    await this.paymentsRepo.save({
      ...result,
      idempotencyKey,
    });

    return result;
  }
}
```

## Common Pitfalls

1. **Retrying non-idempotent operations**: Can cause duplicate effects.
2. **No maximum delay**: Exponential backoff can grow huge.
3. **Retrying permanent failures**: 404s won't succeed on retry.

## Best Practices

- Use exponential backoff with jitter
- Set maximum retry count and delay
- Only retry transient failures (5xx, timeouts)
- Implement circuit breaker for failing services
- Use idempotency keys for critical operations

## Summary

Retry patterns add resilience to external calls. Use exponential backoff with jitter, circuit breakers for failing services, and idempotency for safe retries. Only retry transient failures and set appropriate limits.

## Resources

- [Techniques](https://docs.nestjs.com/techniques) — Advanced techniques

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*