---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-turbopack-intro"
---

# Turbopack

Turbopack is Next.js's new Rust-powered bundler, designed to be the successor to Webpack.

## Enabling Turbopack

```bash
next dev --turbopack
```

Or in your `package.json`:

```json
{
  "scripts": {
    "dev": "next dev --turbopack"
  }
}
```

## Benefits

- **Faster startup**: Up to 10x faster initial compilation.
- **Faster HMR**: Updates in milliseconds.
- **Incremental compilation**: Only rebuilds what changed.

## Current Status (Next.js 15)

- Stable for development (`next dev`).
- Production builds (`next build`) still use Webpack.

## When to Use

Use Turbopack for development to speed up your workflow. Your production builds will still use the stable Webpack pipeline.

## Resources

- [Turbopack Documentation](https://nextjs.org/docs/app/api-reference/turbopack) — Official Turbopack documentation

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*