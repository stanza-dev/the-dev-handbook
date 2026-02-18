---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-memory-optimization"
---

# Memory Optimization

## Introduction

Memory leaks and excessive consumption crash applications and increase costs. Understanding Node.js memory behavior and NestJS patterns helps build efficient, stable applications.

## Key Concepts

- **Heap**: Where objects are allocated
- **Garbage Collection**: Automatic memory reclamation
- **Memory Leak**: Unreleased references preventing GC
- **Request Scoping**: Per-request provider instances

## Deep Dive

### Monitor Memory Usage

```typescript
@Injectable()
export class MemoryService {
  getMemoryUsage() {
    const used = process.memoryUsage();
    return {
      heapUsed: Math.round(used.heapUsed / 1024 / 1024) + ' MB',
      heapTotal: Math.round(used.heapTotal / 1024 / 1024) + ' MB',
      external: Math.round(used.external / 1024 / 1024) + ' MB',
      rss: Math.round(used.rss / 1024 / 1024) + ' MB',
    };
  }
}
```

### Common Leak Sources

```typescript
// BAD: Growing array
@Injectable()
export class LeakyService {
  private cache: any[] = []; // Never cleared!
  
  add(item: any) {
    this.cache.push(item);
  }
}

// GOOD: Bounded cache
@Injectable()
export class BoundedCacheService {
  private cache = new Map<string, any>();
  private maxSize = 1000;
  
  set(key: string, value: any) {
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    this.cache.set(key, value);
  }
}
```

### Request-Scoped Memory

```typescript
// Request scope creates new instance per request
@Injectable({ scope: Scope.REQUEST })
export class RequestDataService {
  private data: any[] = []; // Garbage collected after request
}
```

### Stream Large Data

```typescript
// BAD: Load all into memory
async exportAllUsers(): Promise<User[]> {
  return this.userRepository.find(); // 1M users = OOM
}

// GOOD: Stream
async exportAllUsers(res: Response) {
  const stream = this.userRepository
    .createQueryBuilder('user')
    .stream();
  
  res.setHeader('Content-Type', 'application/json');
  stream.pipe(res);
}
```

### Cleanup Resources

```typescript
@Injectable()
export class ExternalService implements OnModuleDestroy {
  private connections: Connection[] = [];
  
  onModuleDestroy() {
    // Close connections on shutdown
    this.connections.forEach(conn => conn.close());
  }
}
```

## Common Pitfalls

1. **Unbounded caches**: Always set max size and eviction policy.
2. **Event listener leaks**: Remove listeners when no longer needed.
3. **Large request bodies**: Stream instead of buffering.

## Best Practices

- Monitor memory in production
- Use bounded caches with LRU eviction
- Stream large data sets
- Clean up resources in OnModuleDestroy
- Profile with --inspect and Chrome DevTools

## Summary

Memory optimization involves monitoring usage, using bounded caches, streaming large data, and cleaning up resources. Request-scoped providers help isolate per-request data. Profile with Node.js inspector to find leaks.

## Resources

- [Performance](https://docs.nestjs.com/techniques/performance) — Performance tips

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*