---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-parsing-loading"
---

# Parsing & Code Loading

## Introduction

Before code runs, it must be parsed. Large bundles take time to parse, affecting startup performance.

## Deep Dive

### Parsing Cost

```javascript
// Parsing is O(n) in code size
// 1MB of JS ≈ 200-300ms parse time on mobile

// Lazy parsing (pre-parsing)
function notUsedYet() {
  // Engine pre-parses (minimal work)
  // Full parse happens on first call
}
```

### Code Caching

```javascript
// V8 caches compiled code
// Second load of same script is faster
// Service Workers can cache compiled code

// Affected by:
// - Script source must be identical
// - Same origin and cache headers
```

### Module Evaluation

```javascript
// ES modules are parsed before execution
import { heavy } from './heavy.js';

// All imports must be resolved first
// Tree-shaking can reduce parse work

// Dynamic import delays parsing
const module = await import('./lazy.js');
```

### Reducing Parse Time

```javascript
// 1. Code splitting
// Only load what's needed
import('./feature.js').then(module => {
  module.init();
});

// 2. Remove dead code
// Tree-shaking with modern bundlers

// 3. Avoid large inline data
// Don't embed huge JSON in JS
const data = await fetch('/data.json');
```

## Best Practices

- **Split code by route/feature**: Reduce initial parse.
- **Use dynamic imports**: Lazy load non-critical code.
- **Minimize bundle size**: Less code = faster parse.
- **Leverage browser caching**: Avoids re-parsing.

## Summary

Parsing takes time proportional to code size. Lazy parsing defers work. Code splitting and dynamic imports reduce initial parse cost. Browser caching helps repeat visits.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*