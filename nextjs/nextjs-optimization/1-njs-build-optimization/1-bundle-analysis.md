---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-bundle-analysis"
---

# Analyzing Your Bundle

Understanding what's in your JavaScript bundle is the first step to optimization.

## Using @next/bundle-analyzer

```bash
npm install @next/bundle-analyzer
```

```javascript
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // your config
});
```

Run with:
```bash
ANALYZE=true npm run build
```

## What to Look For

1. **Large dependencies**: Are you importing entire libraries when you only need a few functions?
2. **Duplicate packages**: Different versions of the same package.
3. **Unused code**: Code that's imported but never used.

## Common Fixes

```typescript
// Bad: imports entire library
import { format } from 'date-fns';

// Good: tree-shakeable import
import format from 'date-fns/format';
```

## Resources

- [Bundle Analyzer](https://nextjs.org/docs/app/building-your-application/optimizing/bundle-analyzer) — Official bundle analyzer documentation

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*