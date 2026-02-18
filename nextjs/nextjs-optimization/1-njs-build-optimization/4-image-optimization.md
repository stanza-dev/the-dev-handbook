---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-image-optimization"
---

# Image Optimization

## Introduction

Images often account for 50%+ of page weight. Next.js's Image component automatically optimizes images with lazy loading, modern formats, and responsive sizing.

## Key Concepts

**next/image** provides:

- **Automatic format conversion**: WebP/AVIF for supported browsers
- **Responsive images**: Multiple sizes for different viewports
- **Lazy loading**: Images load as they enter viewport
- **Size optimization**: Serve appropriately sized images

## Real World Context

Without image optimization:
- 2MB hero image on mobile (should be 100KB)
- Layout shift as images load
- Poor Core Web Vitals scores
- Slow page loads on 3G connections

## Deep Dive

### Basic Usage

```typescript
import Image from 'next/image';

export function Hero() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero image"
      width={1200}
      height={600}
      priority // Load immediately (above fold)
    />
  );
}
```

### Responsive Images

```typescript
<Image
  src="/hero.jpg"
  alt="Hero"
  fill // Fill parent container
  sizes="(max-width: 768px) 100vw, 50vw"
  style={{ objectFit: 'cover' }}
/>
```

### Remote Images

```typescript
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.example.com',
      },
    ],
  },
};

// Component
<Image
  src="https://images.example.com/photo.jpg"
  alt="Remote image"
  width={800}
  height={600}
/>
```

### Priority and Loading

```typescript
// Above the fold - load immediately
<Image src="/hero.jpg" priority />

// Below the fold - lazy load (default)
<Image src="/gallery-1.jpg" />

// Eager loading (rare)
<Image src="/critical.jpg" loading="eager" />
```

## Common Pitfalls

1. **Missing dimensions**: Without `width`/`height` or `fill`, you get layout shift.

2. **Forgetting `sizes`**: Without `sizes`, responsive images may be larger than needed.

3. **`priority` everywhere**: Only use on above-the-fold images.

## Best Practices

- **Use `priority` for LCP image**: The largest contentful paint image should load first
- **Provide `sizes` for responsive images**: Helps browser choose the right size
- **Use `fill` for unknown dimensions**: With a sized container
- **Configure remote patterns**: For CDN and external images

## Summary

Next.js Image component automatically optimizes images with modern formats, lazy loading, and responsive sizing. Use `priority` for above-fold images, provide `sizes` for responsive images, and configure remote patterns for external sources.

## Resources

- [Image Optimization](https://nextjs.org/docs/app/building-your-application/optimizing/images) — Next.js Image component documentation

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*