---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-monitoring"
---

# Production Monitoring

## Introduction

Deploying is just the beginning. Production monitoring helps you catch issues before users report them, understand performance bottlenecks, and maintain reliability.

## Key Concepts

**Monitoring pillars**:

- **Error tracking**: Catch and diagnose errors
- **Performance monitoring**: Track Core Web Vitals
- **Logging**: Understand what's happening
- **Alerting**: Know when things break

## Deep Dive

### Core Web Vitals Reporting

```typescript
// app/layout.tsx
import { SpeedInsights } from '@vercel/speed-insights/next';
import { Analytics } from '@vercel/analytics/react';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <SpeedInsights />
        <Analytics />
      </body>
    </html>
  );
}
```

### Error Boundaries

```typescript
// app/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  // Log to error tracking service
  useEffect(() => {
    captureException(error);
  }, [error]);

  return (
    <div>
      <h2>Something went wrong</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

### Custom Web Vitals Reporting

```typescript
// app/components/WebVitals.tsx
'use client';

import { useReportWebVitals } from 'next/web-vitals';

export function WebVitals() {
  useReportWebVitals((metric) => {
    // Send to analytics
    analytics.track('Web Vitals', {
      name: metric.name,
      value: metric.value,
      rating: metric.rating,
    });
  });

  return null;
}
```

## Summary

Monitor production apps with error tracking, performance metrics, and logging. Use Vercel Analytics for Core Web Vitals, implement error boundaries, and set up alerting for critical issues.

## Resources

- [Analytics](https://nextjs.org/docs/app/building-your-application/optimizing/analytics) — Analytics and monitoring in Next.js

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*