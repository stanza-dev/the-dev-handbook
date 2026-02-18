---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-profiling"
---

# Performance Profiling

## Introduction

Profilers identify performance bottlenecks by measuring where time is spent. CPU profiling shows slow functions, memory profiling finds leaks, and flame graphs visualize call stacks.

## Key Concepts

- **CPU Profile**: Where is CPU time spent?
- **Heap Snapshot**: What's in memory?
- **Flame Graph**: Visualization of call stacks
- **Async Hooks**: Track async operations

## Real World Context

Profiling diagnoses:
- Slow API endpoints
- Memory leaks
- Blocked event loop
- Inefficient algorithms

## Deep Dive

### Node.js Inspector

```bash
# Start with inspector
node --inspect dist/main.js

# Chrome DevTools: chrome://inspect
```

### CPU Profiling in Production

```typescript
import * as v8Profiler from 'v8-profiler-next';

@Controller('debug')
export class DebugController {
  @Post('profile/start')
  startProfiling() {
    v8Profiler.startProfiling('CPU Profile');
    return { message: 'Profiling started' };
  }

  @Post('profile/stop')
  async stopProfiling(@Res() res: Response) {
    const profile = v8Profiler.stopProfiling('CPU Profile');

    // Export as .cpuprofile file
    profile.export((error, result) => {
      if (error) throw error;
      res.setHeader('Content-Type', 'application/json');
      res.setHeader('Content-Disposition', 'attachment; filename=profile.cpuprofile');
      res.send(result);
      profile.delete();
    });
  }
}
```

### Memory Profiling

```typescript
import * as v8 from 'v8';

@Controller('debug')
export class DebugController {
  @Get('heap-stats')
  getHeapStats() {
    const stats = v8.getHeapStatistics();
    return {
      heapUsed: `${Math.round(stats.used_heap_size / 1024 / 1024)} MB`,
      heapTotal: `${Math.round(stats.total_heap_size / 1024 / 1024)} MB`,
      heapLimit: `${Math.round(stats.heap_size_limit / 1024 / 1024)} MB`,
      external: `${Math.round(stats.external_memory / 1024 / 1024)} MB`,
    };
  }

  @Get('heap-snapshot')
  async getHeapSnapshot(@Res() res: Response) {
    const snapshotStream = v8.writeHeapSnapshot();
    res.setHeader('Content-Type', 'application/octet-stream');
    res.setHeader('Content-Disposition', 'attachment; filename=heap.heapsnapshot');
    // Read and stream the file
  }
}
```

### Request Timing Middleware

```typescript
@Injectable()
export class TimingMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const start = process.hrtime.bigint();

    res.on('finish', () => {
      const end = process.hrtime.bigint();
      const durationMs = Number(end - start) / 1e6;

      if (durationMs > 1000) {
        console.warn(`Slow request: ${req.method} ${req.url} took ${durationMs.toFixed(2)}ms`);
      }
    });

    next();
  }
}
```

### Event Loop Monitoring

```typescript
import { monitorEventLoopDelay } from 'perf_hooks';

@Injectable()
export class EventLoopMonitorService implements OnModuleInit {
  private histogram: ReturnType<typeof monitorEventLoopDelay>;

  onModuleInit() {
    this.histogram = monitorEventLoopDelay({ resolution: 20 });
    this.histogram.enable();
  }

  getEventLoopStats() {
    return {
      min: this.histogram.min / 1e6,
      max: this.histogram.max / 1e6,
      mean: this.histogram.mean / 1e6,
      p50: this.histogram.percentile(50) / 1e6,
      p99: this.histogram.percentile(99) / 1e6,
    };
  }

  @Cron('*/30 * * * * *')
  checkEventLoop() {
    const stats = this.getEventLoopStats();
    if (stats.p99 > 100) {
      this.logger.warn(`Event loop delay high: p99=${stats.p99.toFixed(2)}ms`);
    }
  }
}
```

### Load Testing

```bash
# Using autocannon
npx autocannon -c 100 -d 30 -p 10 http://localhost:3000/api/users

# Using k6
k6 run load-test.js
```

```javascript
// load-test.js (k6)
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 100,
  duration: '30s',
};

export default function () {
  const res = http.get('http://localhost:3000/api/users');
  check(res, { 'status was 200': (r) => r.status === 200 });
  sleep(0.1);
}
```

## Common Pitfalls

1. **Profiling in production**: Use sampling, not continuous.
2. **Ignoring event loop**: Blocking code kills performance.
3. **No baseline**: Profile before and after changes.

## Best Practices

- Profile with realistic load
- Use flame graphs to visualize bottlenecks
- Monitor event loop delay
- Load test before production
- Profile memory over time to find leaks

## Summary

Profiling identifies bottlenecks through CPU profiles, heap snapshots, and event loop monitoring. Use Node.js inspector, load testing tools, and custom timing middleware. Profile with realistic load and compare against baselines.

## Resources

- [Performance](https://docs.nestjs.com/techniques/performance) — Performance optimization

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*