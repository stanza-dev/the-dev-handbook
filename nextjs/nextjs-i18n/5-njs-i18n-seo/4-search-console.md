---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-search-console"
---

# International SEO Monitoring

## Introduction

Setting up hreflang is just the start. You need to monitor how search engines crawl and index your localized pages.

## Key Concepts

**Monitoring areas**:

- Hreflang validation
- Index coverage by locale
- Search performance by country
- Crawl errors

## Deep Dive

### Google Search Console Setup

1. **Add property for each locale subdomain/subfolder**
   - example.com/en/
   - example.com/fr/
   - example.com/de/

2. **Or use URL parameter handling**
   - Configure locale parameters

### Checking Hreflang Implementation

```bash
# Validate hreflang tags
curl -s https://example.com/en | grep -i hreflang
```

Expected output:
```html
<link rel="alternate" hreflang="en" href="https://example.com/en" />
<link rel="alternate" hreflang="fr" href="https://example.com/fr" />
<link rel="alternate" hreflang="x-default" href="https://example.com/en" />
```

### Common Hreflang Errors

- Missing return links (A→B but not B→A)
- Wrong language codes (en-uk vs en-GB)
- Missing self-referential hreflang
- x-default pointing to non-existent page

### Performance Monitoring

```typescript
// Track locale in analytics
const locale = params.lang;

analytics.track('page_view', {
  locale,
  path: pathname,
  country: geo?.country,
});
```

## Summary

Monitor international SEO through Search Console, validate hreflang implementation, and track performance by locale. Fix return link errors promptly as they can hurt rankings.

## Resources

- [International SEO](https://developers.google.com/search/docs/specialty/international) — Google's international SEO guide

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*