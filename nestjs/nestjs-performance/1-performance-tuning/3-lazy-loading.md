---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-lazy-loading"
---

# Lazy Loading Modules

## Introduction

Not all features are used by all users. Lazy loading defers module initialization until needed, reducing startup time and memory usage. This is especially valuable for large applications with many optional features.

## Key Concepts

- **LazyModuleLoader**: Service for dynamic module loading
- **Cold Start**: Initial application startup time
- **Code Splitting**: Loading code on demand vs upfront

## Real World Context

Imagine a SaaS application with an admin dashboard that uses Puppeteer for PDF report generation. Loading Puppeteer adds 200ms to cold start for every request, even though only 2% of users are admins. Lazy loading this module eliminates the penalty for the other 98%.

## Deep Dive

### Setup

```typescript
import { LazyModuleLoader } from '@nestjs/core';

@Injectable()
export class FeatureService {
  constructor(private lazyModuleLoader: LazyModuleLoader) {}
}
```

### Loading a Module on Demand

```typescript
@Injectable()
export class ReportService {
  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  async generateReport(): Promise<Report> {
    const { ReportModule } = await import('./report/report.module');
    const moduleRef = await this.lazyModuleLoader.load(() => ReportModule);
    const reportGenerator = moduleRef.get(ReportGeneratorService);
    return reportGenerator.generate();
  }
}
```

### Lazy Loading Pattern

```typescript
@Injectable()
export class PdfService {
  private pdfModule: ModuleRef | null = null;

  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  private async loadPdfModule(): Promise<ModuleRef> {
    if (!this.pdfModule) {
      const { PdfModule } = await import('./pdf/pdf.module');
      this.pdfModule = await this.lazyModuleLoader.load(() => PdfModule);
    }
    return this.pdfModule;
  }

  async generatePdf(content: string): Promise<Buffer> {
    const moduleRef = await this.loadPdfModule();
    const generator = moduleRef.get(PdfGeneratorService);
    return generator.create(content);
  }
}
```

### Use Cases

- **Admin features**: Only load when admin users access
- **Heavy libraries**: PDF generation, image processing
- **Reporting modules**: Complex analytics and exports
- **Rarely-used integrations**: Third-party service connectors

## Common Pitfalls

1. **First-request latency**: Initial load of lazy module takes time.
2. **Not caching**: Repeatedly loading adds overhead—cache the moduleRef.
3. **Lifecycle hooks not invoked**: `OnModuleInit`, `OnModuleDestroy`, and other lifecycle hooks are NOT called in lazy-loaded modules and services.
4. **Controllers and gateways ignored**: Controllers, resolvers, and WebSocket gateways registered inside lazy-loaded modules will not behave as expected — they won't register routes or listeners.

## Best Practices

- Cache loaded modules for subsequent requests
- Pre-warm frequently-used lazy modules during idle time
- Monitor first-request latency for lazy routes
- Use for genuinely optional, heavy features
- Only lazy-load service-based modules (not modules with controllers or gateways)

## Summary

Lazy loading with LazyModuleLoader defers module initialization until needed. This reduces startup time and memory for features not all users need. Cache loaded modules and pre-warm critical lazy features.

## Code Examples

**Lazy loading a module on demand — the module is only initialized when generateReport() is first called**

```typescript
import { LazyModuleLoader } from '@nestjs/core';

@Injectable()
export class ReportService {
  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  async generateReport(): Promise<Report> {
    const { ReportModule } = await import('./report/report.module');
    const moduleRef = await this.lazyModuleLoader.load(() => ReportModule);
    const reportGenerator = moduleRef.get(ReportGeneratorService);
    return reportGenerator.generate();
  }
}
```


## Resources

- [Lazy Loading](https://docs.nestjs.com/fundamentals/lazy-loading-modules) — Official lazy loading guide

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*