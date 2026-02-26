---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-devtools"
---

## Introduction

Chrome DevTools Performance panel is the most powerful tool available for diagnosing JavaScript performance problems in the browser. It records a timeline of everything the browser does — JavaScript execution, style calculation, layout, paint, compositing, garbage collection, and network requests — and presents it as an interactive flame chart that shows exactly where time is being spent.

Knowing how to record a profile, read the flame chart, identify long tasks, and use CPU throttling turns performance optimization from guesswork into science.

## Key Concepts

- **Performance Panel**: The Chrome DevTools tab that records and visualizes a timeline of browser activity. It shows CPU usage, frames per second, network requests, and a detailed breakdown of main thread activity.
- **Flame Chart**: A visualization where each horizontal bar represents a function call, and nested bars represent calls made by that function. The width of each bar corresponds to its execution time. Tall, wide stacks indicate expensive call chains.
- **Main Thread**: The single thread where JavaScript, style calculation, layout, and paint run. Long tasks on the main thread block user input and cause jank. The Performance panel shows main thread activity as the primary timeline.
- **Long Tasks**: Any task that occupies the main thread for more than 50ms. Long tasks are highlighted with a red flag in the Performance panel and are the primary cause of input delay and visual jank.
- **Web Vitals**: Google's standardized metrics for user experience: Largest Contentful Paint (LCP) measures loading, Interaction to Next Paint (INP) measures responsiveness, and Cumulative Layout Shift (CLS) measures visual stability. The Performance panel overlays these metrics on the timeline.

## Real World Context

Performance profiling is a daily activity at companies like Google, Meta, and Netflix. When Netflix noticed increased INP on their browse page, DevTools profiling revealed that a complex intersection observer callback was running synchronously for all visible title cards, creating a 200ms long task. Breaking it into chunked microtasks reduced INP by 80%. The Performance panel's flame chart is also used to diagnose memory leaks by correlating GC pauses with increasing heap size. Chrome's Lighthouse audit internally uses the same profiling infrastructure to generate its performance score.

## Deep Dive

To record a performance profile, open DevTools (F12), navigate to the Performance tab, and click the record button (or press Ctrl+E / Cmd+E). Interact with the page to capture the behavior you want to analyze, then stop recording.

The resulting timeline shows several lanes:
- **Frames**: Green bars show each rendered frame. Short bars mean smooth rendering, while gaps or tall bars indicate dropped frames.
- **Main**: The flame chart of main thread activity. JavaScript execution, style recalculation, layout, paint, and GC are all shown here.
- **Network**: Resource loading timeline.
- **Timings**: User Timing marks and measures, plus Web Vitals markers.

You can also trigger profiling programmatically:

```javascript
console.profile('my-operation');
performHeavyWork();
console.profileEnd('my-operation');
```

Long Task detection can be done in code using PerformanceObserver, allowing you to monitor for jank in production:

```javascript
const observer = new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    console.warn(`Long task: ${entry.duration}ms`, {
      startTime: entry.startTime,
      name: entry.name,
    });
  }
});
observer.observe({ type: 'longtask', buffered: true });
```

Web Vitals can also be monitored programmatically:

```javascript
// Largest Contentful Paint
new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    console.log('LCP:', entry.startTime, 'ms');
    console.log('LCP element:', entry.element);
  }
}).observe({ type: 'largest-contentful-paint', buffered: true });

// Layout Shift
new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    if (!entry.hadRecentInput) {
      console.log('CLS shift:', entry.value);
    }
  }
}).observe({ type: 'layout-shift', buffered: true });
```

CPU throttling is essential for realistic profiling. Most developers work on high-end machines, but their users may be on mid-range mobile devices. In the Performance panel, use the gear icon to set CPU throttling to 4x or 6x slowdown. This simulates the experience of users on slower devices and reveals performance problems that are invisible on fast hardware.

The Bottom-Up tab in the Performance panel aggregates time spent across all instances of each function, revealing which functions consume the most total time even if no single call is particularly long. The Call Tree tab shows the same data organized by call hierarchy.

## Common Pitfalls

- **Profiling without CPU throttling**: Your development machine is likely 4-10x faster than your average user's device. Always enable CPU throttling when profiling to catch performance issues that affect real users.
- **Recording too much data**: Recording for more than 10-15 seconds produces enormous traces that are slow to analyze. Focus on specific interactions: record, perform the action, stop immediately.
- **Ignoring minor GC pauses**: Frequent small garbage collection pauses that individually seem insignificant can accumulate into noticeable jank. Look for regular GC activity patterns that indicate excessive object allocation.

## Best Practices

- Profile specific user interactions rather than recording the entire page load. Click record, perform exactly one interaction (click, scroll, type), and stop immediately.
- Use the PerformanceObserver long task API in production to detect and report jank that only occurs with real user data and network conditions.
- Compare flame charts before and after optimization to verify that your changes actually reduced the work being done, rather than just moving it around.

## Summary

Chrome DevTools Performance panel records a timeline of all browser activity and presents it as an interactive flame chart. Long tasks (over 50ms) are the primary cause of jank and are flagged in the timeline. CPU throttling simulates slower devices for realistic profiling. Use PerformanceObserver in production to detect long tasks and monitor Web Vitals. Always profile specific interactions rather than recording entire sessions, and use the Bottom-Up view to find the most expensive functions across the entire recording.

## Code Examples

**Programmatic profiling and Long Task detection**

```javascript
// Programmatic performance recording
console.profile('my-operation');
performHeavyWork();
console.profileEnd('my-operation');

// Long Task detection
const observer = new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    console.warn(`Long task: ${entry.duration}ms`, {
      startTime: entry.startTime,
      name: entry.name,
    });
  }
});
observer.observe({ type: 'longtask', buffered: true });

// Web Vitals monitoring
new PerformanceObserver(list => {
  for (const entry of list.getEntries()) {
    console.log('LCP:', entry.startTime);
  }
}).observe({ type: 'largest-contentful-paint', buffered: true });
```


## Resources

- [Chrome DevTools: Performance](https://developer.chrome.com/docs/devtools/performance) — Chrome DevTools Performance panel guide

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*