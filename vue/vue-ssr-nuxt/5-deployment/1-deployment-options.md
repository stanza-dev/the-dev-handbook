---
source_course: "vue-ssr-nuxt"
source_lesson: "vue-ssr-nuxt-deployment-options"
---

# Deployment Options

Nuxt can be deployed in various ways depending on your rendering strategy.

## Build Commands

```bash
# SSR (Server-Side Rendering)
npm run build
# Creates .output/ directory

# SSG (Static Site Generation)
npm run generate
# Creates .output/public/ directory
```

## Node.js Server (SSR)

```bash
# Build
npm run build

# Start production server
node .output/server/index.mjs
# or
npm run preview
```

### PM2 Process Manager

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'nuxt-app',
    script: '.output/server/index.mjs',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    }
  }]
}
```

```bash
pm2 start ecosystem.config.js
```

## Static Hosting (SSG)

```bash
# Generate static files
npm run generate

# Output in .output/public/
# Deploy to any static host
```

Works with:
- Netlify
- Vercel (static)
- GitHub Pages
- Cloudflare Pages
- AWS S3 + CloudFront

## Vercel

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel
```

Or connect GitHub repo to Vercel dashboard.

```json
// vercel.json (optional)
{
  "buildCommand": "npm run build",
  "outputDirectory": ".output"
}
```

## Netlify

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = ".output/public"

[build.environment]
  NODE_VERSION = "18"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

For SSR:

```toml
[build]
  command = "npm run build"

[functions]
  directory = ".output/server"
```

## Cloudflare Pages

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare-pages'
  }
})
```

## Docker

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci

# Copy source
COPY . .

# Build
RUN npm run build

# Expose port
EXPOSE 3000

# Start server
CMD ["node", ".output/server/index.mjs"]
```

```bash
# Build and run
docker build -t nuxt-app .
docker run -p 3000:3000 nuxt-app
```

## Environment Variables

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    // Server only
    apiSecret: process.env.API_SECRET,
    
    // Client + Server
    public: {
      apiBase: process.env.API_BASE || '/api'
    }
  }
})
```

```vue
<!-- In components -->
<script setup>
const config = useRuntimeConfig()

// Client + Server
console.log(config.public.apiBase)

// Server only (in server/ directory)
console.log(config.apiSecret)
</script>
```

## Pre-rendering Specific Routes

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    prerender: {
      routes: ['/sitemap.xml', '/robots.txt'],
      crawlLinks: true
    }
  },
  
  routeRules: {
    '/': { prerender: true },
    '/blog/**': { isr: 3600 },
    '/dashboard/**': { ssr: false }
  }
})
```

## Health Checks

```typescript
// server/api/health.get.ts
export default defineEventHandler(() => {
  return {
    status: 'healthy',
    timestamp: new Date().toISOString()
  }
})
```

## Performance Considerations

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Experimental features
  experimental: {
    payloadExtraction: true,
    renderJsonPayloads: true
  },
  
  // Compression
  nitro: {
    compressPublicAssets: true
  },
  
  // App config
  app: {
    head: {
      link: [
        { rel: 'preconnect', href: 'https://api.example.com' }
      ]
    }
  }
})
```

## Resources

- [Nuxt Deployment](https://nuxt.com/docs/getting-started/deployment) — Nuxt deployment guide

---

> 📘 *This lesson is part of the [Vue Server-Side Rendering & Nuxt](https://stanza.dev/courses/vue-ssr-nuxt) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*