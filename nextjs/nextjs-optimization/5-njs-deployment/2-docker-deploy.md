---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-docker-deploy"
---

# Self-Hosting Next.js

You can deploy Next.js anywhere that supports Node.js.

## Standalone Build

Enable the standalone output mode:

```javascript
// next.config.js
module.exports = {
  output: 'standalone',
};
```

This creates a minimal production bundle at `.next/standalone`.

## Dockerfile

```dockerfile
FROM node:18-alpine AS base

FROM base AS builder
WORKDIR /app
COPY . .
RUN npm ci
RUN npm run build

FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public

EXPOSE 3000
ENV PORT 3000
CMD ["node", "server.js"]
```

## Running Without Docker

```bash
npm run build
npm run start
```

This starts the Next.js production server on port 3000.

## Static Export

For purely static sites:

```javascript
// next.config.js
module.exports = {
  output: 'export',
};
```

This generates an `out/` directory with static HTML.

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*