---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-turbopack-migration"
---

# Migrating to Turbopack

## Introduction

Turbopack offers significant speed improvements, but migrating requires checking compatibility with your current Webpack setup. Here's how to migrate safely.

## Key Concepts

**Migration considerations**:

- **Webpack loaders**: Some need Turbopack equivalents
- **Custom webpack config**: May need adjustments
- **Third-party packages**: Most work, some don't

## Deep Dive

### Testing Compatibility

```bash
# Try Turbopack in development first
next dev --turbopack

# Watch for errors or missing features
```

### Common Adjustments

```javascript
// next.config.js
module.exports = {
  experimental: {
    turbo: {
      // Webpack loader compatibility
      rules: {
        '*.svg': {
          loaders: ['@svgr/webpack'],
          as: '*.js',
        },
      },
      // Module resolution
      resolveAlias: {
        canvas: './empty-module.js',
      },
    },
  },
};
```

### Performance Comparison

```bash
# Webpack development
time next dev
# Cold start: ~15s, HMR: ~1s

# Turbopack development  
time next dev --turbopack
# Cold start: ~3s, HMR: ~50ms
```

## Common Pitfalls

1. **Assuming all loaders work**: Test thoroughly before committing.
2. **Skipping the compatibility check**: Some features silently fail.

## Summary

Migrate to Turbopack gradually—start with development, test thoroughly, and configure loaders as needed. The speed improvements are worth the migration effort.

## Resources

- [Turbopack Compatibility](https://nextjs.org/docs/app/api-reference/turbopack) — Turbopack feature compatibility

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*