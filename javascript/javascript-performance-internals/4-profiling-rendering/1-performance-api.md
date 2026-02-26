---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-performance-api"
---

## Introduction

Before you can optimize, you must measure. Intuition about what is slow is frequently wrong — developers routinely optimize the wrong code path because they guessed instead of profiled. The browser's Performance API provides sub-millisecond timing precision, named markers for annotating your code, and observer patterns for continuous monitoring, all without any external libraries.

This lesson covers the User Timing API for custom measurements, PerformanceObserver for automated monitoring, and the navigation and resource timing APIs for understanding page load performance.

## Key Concepts

- **performance.now()**: Returns a high-resolution timestamp in milliseconds (with microsecond precision) relative to the page's navigation start. Unlike `Date.now()`, it is monotonic (never goes backward) and unaffected by system clock adjustments.
- **performance.mark()**: Creates a named timestamp in the performance timeline. Marks serve as reference points that you can measure between.
- **performance.measure()**: Creates a named duration measurement between two marks (or from navigation start to a mark). The resulting entry includes the duration, start time, and name.
- **PerformanceObserver**: An API for asynchronously observing performance entries as they are recorded. It can observe marks, measures, long tasks, resource loads, paint events, and Web Vitals metrics.
- **User Timing API**: The collective name for `performance.mark()` and `performance.measure()`, providing developers with a standardized way to instrument their code with custom timing data.

## Real World Context

Production applications use the Performance API to collect Real User Monitoring (RUM) data. Companies like Google, Netflix, and Airbnb send performance measurements to analytics services to track Core Web Vitals (LCP, FID, CLS) across their user base. The User Timing API integrates directly with Chrome DevTools — marks and measures appear as labeled annotations in the Performance panel's timeline, making it easy to correlate custom measurements with browser activity. Server-side rendering frameworks use navigation timing to measure Time to First Byte (TTFB) and hydration duration.

## Deep Dive

The simplest measurement uses `performance.now()` for high-resolution timestamps:

```javascript
const start = performance.now();
expensiveOperation();
const duration = performance.now() - start;
console.log(`Operation took ${duration.toFixed(2)}ms`);
```

For more structured measurement, use marks and measures:

```javascript
performance.mark('render-start');
renderComponent();
performance.mark('render-end');
performance.measure('render-time', 'render-start', 'render-end');

const [entry] = performance.getEntriesByName('render-time');
console.log(`Render took ${entry.duration.toFixed(2)}ms`);
```

PerformanceObserver provides a non-blocking way to monitor entries as they are recorded, including buffered entries from before the observer was created:

```javascript
const observer = new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 50) {
      console.warn('Slow task:', entry.name, entry.duration);
    }
  }
});
observer.observe({ type: 'measure', buffered: true });
```

You can observe multiple entry types for comprehensive monitoring:

```javascript
// Monitor resource loading
new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 1000) {
      console.warn(`Slow resource: ${entry.name} (${entry.duration}ms)`);
    }
  }
}).observe({ type: 'resource', buffered: true });

// Monitor navigation timing
new PerformanceObserver(list => {
  const [nav] = list.getEntries();
  console.log('TTFB:', nav.responseStart - nav.requestStart);
  console.log('DOM Interactive:', nav.domInteractive);
  console.log('DOM Complete:', nav.domComplete);
}).observe({ type: 'navigation', buffered: true });
```

For production monitoring, send measurements to your analytics endpoint:

```javascript
function reportMetric(name, value) {
  navigator.sendBeacon('/analytics', JSON.stringify({ name, value, url: location.href }));
}

new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    reportMetric(entry.name, entry.duration);
  }
}).observe({ type: 'measure', buffered: true });
```

## Common Pitfalls

- **Using Date.now() for performance measurement**: `Date.now()` only has millisecond precision, is not monotonic (can jump backward due to clock synchronization), and includes time spent in sleep/hibernation. Always use `performance.now()` for timing.
- **Forgetting the `buffered: true` option**: Without `buffered: true`, PerformanceObserver only captures entries created after the observer is registered. Entries recorded during page load will be missed.
- **Not cleaning up observers**: In single-page applications, PerformanceObservers created in components should be disconnected when the component unmounts to prevent memory leaks and duplicate reporting.

## Best Practices

- Use descriptive mark names that include the component or feature being measured (e.g., `'dashboard-render-start'` rather than `'start'`) so they are identifiable in DevTools.
- Always use `buffered: true` when creating PerformanceObservers to capture entries that were recorded before your observer code ran, especially for page load metrics.
- Send performance data using `navigator.sendBeacon()` rather than `fetch()` to ensure data is transmitted even during page unload.

## Summary

The Performance API provides high-resolution timing with `performance.now()`, structured measurement with marks and measures, and asynchronous monitoring with PerformanceObserver. These tools integrate directly with Chrome DevTools and form the foundation of Real User Monitoring. Use `navigator.sendBeacon()` to reliably transmit performance data to analytics services, and always prefer `performance.now()` over `Date.now()` for timing measurements.

## Code Examples

**Using Performance API marks and measures with PerformanceObserver**

```javascript
// Mark and measure
performance.mark('render-start');
renderComponent();
performance.mark('render-end');
performance.measure('render-time', 'render-start', 'render-end');

const [entry] = performance.getEntriesByName('render-time');
console.log(`Render took ${entry.duration.toFixed(2)}ms`);

// PerformanceObserver for continuous monitoring
const observer = new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 50) {
      console.warn('Slow task:', entry.name, entry.duration);
    }
  }
});
observer.observe({ type: 'measure', buffered: true });
```


## Resources

- [MDN: Performance API](https://developer.mozilla.org/en-US/docs/Web/API/Performance_API) — Browser Performance API for precise timing measurements

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*