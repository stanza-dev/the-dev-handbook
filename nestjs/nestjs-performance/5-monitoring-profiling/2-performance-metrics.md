---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-metrics"
---

# Application Metrics

## Introduction

Metrics quantify application behavior—request rates, response times, error counts. They enable dashboards, alerts, and capacity planning. Prometheus and Grafana are the industry standard.

## Key Concepts

- **Counter**: Cumulative metric (requests, errors)
- **Gauge**: Point-in-time value (connections, queue size)
- **Histogram**: Distribution of values (response times)
- **Prometheus**: Metrics collection system

## Real World Context

Metrics answer:
- How many requests per second?
- What's the 95th percentile response time?
- How many errors occurred?
- Is memory usage growing?

## Deep Dive

### Setup with Prometheus

```bash
npm install prom-client
```

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService implements OnModuleInit {
  private registry: client.Registry;
  public httpRequestsTotal: client.Counter;
  public httpRequestDuration: client.Histogram;
  public activeConnections: client.Gauge;

  onModuleInit() {
    this.registry = new client.Registry();

    // Default metrics (memory, CPU, etc.)
    client.collectDefaultMetrics({ register: this.registry });

    // Custom metrics
    this.httpRequestsTotal = new client.Counter({
      name: 'http_requests_total',
      help: 'Total number of HTTP requests',
      labelNames: ['method', 'path', 'status'],
      registers: [this.registry],
    });

    this.httpRequestDuration = new client.Histogram({
      name: 'http_request_duration_seconds',
      help: 'HTTP request duration in seconds',
      labelNames: ['method', 'path', 'status'],
      buckets: [0.01, 0.05, 0.1, 0.5, 1, 5],
      registers: [this.registry],
    });

    this.activeConnections = new client.Gauge({
      name: 'active_connections',
      help: 'Number of active connections',
      registers: [this.registry],
    });
  }

  async getMetrics(): Promise<string> {
    return this.registry.metrics();
  }
}
```

### Metrics Interceptor

```typescript
@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  constructor(private metricsService: MetricsService) {}

  intercept(context: ExecutionContext, next: CallHandler) {
    const request = context.switchToHttp().getRequest();
    const { method, route } = request;
    const path = route?.path || request.url;
    const start = Date.now();

    return next.handle().pipe(
      tap(() => {
        const response = context.switchToHttp().getResponse();
        const duration = (Date.now() - start) / 1000;

        this.metricsService.httpRequestsTotal.inc({
          method,
          path,
          status: response.statusCode,
        });

        this.metricsService.httpRequestDuration.observe(
          { method, path, status: response.statusCode },
          duration,
        );
      }),
    );
  }
}
```

### Metrics Endpoint

```typescript
@Controller('metrics')
export class MetricsController {
  constructor(private metricsService: MetricsService) {}

  @Get()
  async getMetrics(@Res() response: Response) {
    response.set('Content-Type', 'text/plain');
    response.send(await this.metricsService.getMetrics());
  }
}
```

### Business Metrics

```typescript
@Injectable()
export class OrdersService {
  private ordersCreated: client.Counter;
  private orderValue: client.Histogram;

  constructor(private metricsService: MetricsService) {
    this.ordersCreated = new client.Counter({
      name: 'orders_created_total',
      help: 'Total orders created',
      labelNames: ['status'],
    });

    this.orderValue = new client.Histogram({
      name: 'order_value_dollars',
      help: 'Order value distribution',
      buckets: [10, 50, 100, 500, 1000],
    });
  }

  async createOrder(dto: CreateOrderDto): Promise<Order> {
    const order = await this.ordersRepo.save(dto);

    this.ordersCreated.inc({ status: 'success' });
    this.orderValue.observe(order.total);

    return order;
  }
}
```

### Prometheus Configuration

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'nestjs-app'
    static_configs:
      - targets: ['app:3000']
    metrics_path: '/metrics'
    scrape_interval: 15s
```

## Common Pitfalls

1. **High cardinality labels**: Don't use user IDs as labels.
2. **Missing metrics**: Track errors, not just successes.
3. **No aggregation**: Use histograms for latency, not averages.

## Best Practices

- Track request rate, error rate, and latency
- Use histograms for response time percentiles
- Add business metrics (orders, signups)
- Keep label cardinality low
- Set up alerts on key metrics

## Summary

Metrics quantify application behavior with counters, gauges, and histograms. Use prom-client for Prometheus integration, interceptors for HTTP metrics, and custom metrics for business KPIs. Monitor dashboards and set alerts.

## Resources

- [Performance](https://docs.nestjs.com/techniques/performance) — Performance monitoring

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*