---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-distributed-tracing"
---

# Distributed Tracing

## Introduction

In microservices, a single request spans multiple services. Distributed tracing tracks requests across service boundaries, showing latency at each hop and identifying slow services.

## Key Concepts

- **Trace**: End-to-end request journey
- **Span**: Single operation within a trace
- **Context Propagation**: Passing trace ID between services
- **OpenTelemetry**: Standard for observability

## Real World Context

Distributed tracing answers:
- Which service is slow?
- Where did the error occur?
- What's the request flow?
- What are the dependencies?

## Deep Dive

### OpenTelemetry Setup

```bash
npm install @opentelemetry/sdk-node @opentelemetry/auto-instrumentations-node @opentelemetry/exporter-jaeger
```

```typescript
// tracing.ts (load before app)
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';

const sdk = new NodeSDK({
  serviceName: 'nestjs-app',
  traceExporter: new JaegerExporter({
    endpoint: 'http://jaeger:14268/api/traces',
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();

process.on('SIGTERM', () => {
  sdk.shutdown().finally(() => process.exit(0));
});
```

```typescript
// main.ts
import './tracing';  // Import first!
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

### Custom Spans

```typescript
import { trace, context, SpanStatusCode } from '@opentelemetry/api';

@Injectable()
export class OrdersService {
  private tracer = trace.getTracer('orders-service');

  async createOrder(dto: CreateOrderDto): Promise<Order> {
    return this.tracer.startActiveSpan('createOrder', async (span) => {
      try {
        span.setAttribute('order.customer_id', dto.customerId);
        span.setAttribute('order.item_count', dto.items.length);

        // Nested span for database operation
        const order = await this.tracer.startActiveSpan('db.insert', async (dbSpan) => {
          const result = await this.ordersRepo.save(dto);
          dbSpan.end();
          return result;
        });

        // Nested span for external call
        await this.tracer.startActiveSpan('payment.process', async (paymentSpan) => {
          await this.paymentService.charge(order);
          paymentSpan.end();
        });

        span.setStatus({ code: SpanStatusCode.OK });
        return order;
      } catch (error) {
        span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
        span.recordException(error);
        throw error;
      } finally {
        span.end();
      }
    });
  }
}
```

### Context Propagation in HTTP Calls

```typescript
import { context, propagation } from '@opentelemetry/api';

@Injectable()
export class ExternalApiService {
  constructor(private httpService: HttpService) {}

  async callExternalService(data: any): Promise<any> {
    // Extract current context
    const headers: Record<string, string> = {};
    propagation.inject(context.active(), headers);

    // Pass trace context to downstream service
    return firstValueFrom(
      this.httpService.post('https://external-api.com/endpoint', data, {
        headers,
      }),
    );
  }
}
```

### Correlation ID Middleware

```typescript
import { trace, context } from '@opentelemetry/api';

@Injectable()
export class CorrelationMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const span = trace.getActiveSpan();
    const traceId = span?.spanContext().traceId || uuid();

    req['correlationId'] = traceId;
    res.setHeader('X-Correlation-ID', traceId);

    next();
  }
}
```

### Trace-Aware Logging

```typescript
import { trace } from '@opentelemetry/api';

@Injectable()
export class LoggerService {
  log(message: string, context?: Record<string, any>) {
    const span = trace.getActiveSpan();
    const traceContext = span?.spanContext();

    console.log(JSON.stringify({
      timestamp: new Date().toISOString(),
      message,
      traceId: traceContext?.traceId,
      spanId: traceContext?.spanId,
      ...context,
    }));
  }
}
```

## Common Pitfalls

1. **Missing context propagation**: Traces break without header propagation.
2. **Too many spans**: Keep spans meaningful, not for every function.
3. **No sampling**: 100% tracing in production is expensive.

## Best Practices

- Use OpenTelemetry for vendor-neutral tracing
- Propagate context in all inter-service calls
- Add meaningful attributes to spans
- Sample traces in production (1-10%)
- Correlate logs with trace IDs

## Summary

Distributed tracing tracks requests across services. Use OpenTelemetry for instrumentation, Jaeger/Zipkin for visualization. Propagate context in HTTP headers and correlate logs with trace IDs for complete observability.

## Resources

- [Logger](https://docs.nestjs.com/techniques/logger) — Logging and observability

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*