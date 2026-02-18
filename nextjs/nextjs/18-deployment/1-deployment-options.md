---
source_course: "nextjs"
source_lesson: "nextjs-deployment-options"
---

# Deployment

Next.js can be deployed to any hosting provider that supports Node.js or Docker, or as static files.

## Vercel

The creators of Next.js provide **Vercel**, a zero-configuration deployment platform. It supports all Next.js features out of the box (Edge, ISR, Image Optimization).

## Static HTML Export

Next.js can output a purely static application using `output: 'export'` in `next.config.js`. This is useful for hosting on S3, Nginx, or GitHub Pages.

```js
// next.config.js
const nextConfig = {
  output: 'export',
}
module.exports = nextConfig
```

Note: API Routes and Middleware (Proxy) are typically disabled in static exports.

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*