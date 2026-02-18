---
source_course: "nextjs"
source_lesson: "nextjs-instrumentation-setup"
---

# Instrumentation

Next.js provides an `instrumentation.ts` (or `.js`) file to integrate monitoring tools like OpenTelemetry.

This file exports a `register` function that is called **once** when a new Next.js server instance starts.

```ts
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./instrumentation-node')
  }
}
```

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*