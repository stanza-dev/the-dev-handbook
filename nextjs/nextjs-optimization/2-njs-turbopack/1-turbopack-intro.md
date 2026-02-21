---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-turbopack-intro"
---

# Turbopack

Turbopack is Next.js's new Rust-powered bundler, designed to be the successor to Webpack.

## Turbopack is the Default

In Next.js 16, Turbopack is the default bundler for both development and production builds. No flag is needed—it just works.

To opt out and use Webpack instead:

```bash
next dev --webpack
next build --webpack
```

## Benefits

- **Faster startup**: Up to 10x faster initial compilation.
- **Faster HMR**: Updates in milliseconds.
- **Incremental compilation**: Only rebuilds what changed.

## Current Status (Next.js 16)

- Turbopack is the default bundler for both `next dev` and `next build`.
- Use the `--webpack` flag to opt out if needed.

## When to Use Webpack

Most projects should use Turbopack (the default). Use `--webpack` only if you have Webpack-specific plugins or configurations that are not yet supported by Turbopack.

## Resources

- [Turbopack Documentation](https://nextjs.org/docs/app/api-reference/turbopack) — Official Turbopack documentation

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*