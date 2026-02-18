---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-dev-performance"
---

# Development Performance Tips

## Introduction

Slow development servers kill productivity. Beyond Turbopack, there are several techniques to speed up your development environment.

## Key Concepts

**Dev performance factors**:

- **TypeScript checking**: Can slow builds significantly
- **File watching**: Too many files = slow HMR
- **Dependencies**: Large node_modules = slow startup

## Deep Dive

### Separate Type Checking

```json
// package.json
{
  "scripts": {
    "dev": "next dev",
    "typecheck": "tsc --noEmit --watch"
  }
}
```

### Ignore Unnecessary Files

```javascript
// next.config.js
module.exports = {
  experimental: {
    // Ignore large directories during dev
    outputFileTracingIgnores: ['./node_modules/@swc/**/*'],
  },
};
```

### Use .env.local Wisely

```bash
# Avoid fetching external data during dev startup
DISABLE_ANALYTICS=true
USE_MOCK_DATA=true
```

## Summary

Optimize development by separating type checking, ignoring unnecessary files, and using mock data during development. Combined with Turbopack, these techniques keep your dev environment fast.

## Resources

- [Development Performance](https://nextjs.org/docs/app/building-your-application/optimizing/development) — Optimizing development performance

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*