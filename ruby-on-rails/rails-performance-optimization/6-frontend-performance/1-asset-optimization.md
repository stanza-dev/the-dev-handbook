---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-asset-optimization"
---

# Asset Optimization

## Introduction
Frontend assets — CSS, JavaScript, and images — significantly impact page load times. Rails 8 uses Propshaft as the default asset pipeline and Thruster for HTTP compression in production, replacing the older Sprockets pipeline.

## Key Concepts
- **Propshaft**: Rails 8's default asset pipeline — simpler than Sprockets. It fingerprints assets for cache busting but doesn't compile or compress them.
- **Thruster**: A lightweight HTTP proxy included in Rails 8's production Dockerfile that handles gzip compression, X-Sendfile, and asset caching headers.
- **Import Maps**: Rails' default JavaScript approach — maps ES module imports to CDN or local URLs without a bundler.

## Real World Context
A typical Rails 8 app serves assets through Thruster in production, which automatically compresses responses and sets proper cache headers. No webpack, no esbuild, no build step — just Propshaft for fingerprinting and Thruster for delivery.

## Deep Dive

### Propshaft (Rails 8 Default)

Propshaft fingerprints assets for cache busting but doesn't compile them:

```ruby
# No compressor configuration needed!
# Propshaft just fingerprints: application.css → application-abc123.css

# config/environments/production.rb
config.assets.css_compressor = nil  # Propshaft doesn't use compressors
# Thruster handles compression at the HTTP level
```

### Thruster in Production

The Rails 8 Dockerfile uses Thruster automatically:

```dockerfile
# Default Rails 8 Dockerfile CMD
CMD ["./bin/thrust", "./bin/rails", "server"]
```

Thruster provides:
- Gzip/Brotli compression for all responses
- X-Sendfile for efficient file serving
- Cache headers for fingerprinted assets

### Import Maps

```ruby
# config/importmap.rb
pin 'application', preload: true
pin '@hotwired/turbo-rails', to: 'turbo.min.js', preload: true
pin '@hotwired/stimulus', to: 'stimulus.min.js', preload: true
```

### Image Optimization with Active Storage

```ruby
class Product < ApplicationRecord
  has_one_attached :image

  def thumbnail
    image.variant(resize_to_limit: [200, 200]).processed
  end
end
```

```erb
<%= image_tag @product.thumbnail, loading: 'lazy' %>
```

### CDN Configuration

```ruby
# config/environments/production.rb
config.asset_host = 'https://cdn.example.com'
```

## Common Pitfalls
1. **Adding Sprockets configuration to a Propshaft app** — `config.assets.css_compressor` and `config.assets.js_compressor` are Sprockets settings. Propshaft doesn't use them.
2. **Not using lazy loading for images** — Below-the-fold images should use `loading: 'lazy'` to defer loading until the user scrolls.

## Best Practices
1. **Let Thruster handle compression** — Don't add middleware for gzip compression. Thruster handles it automatically in production.
2. **Use Active Storage variants for responsive images** — Serve appropriately sized images instead of full-resolution originals.

## Summary
- Rails 8 uses Propshaft for asset fingerprinting — no compilation or compression config needed.
- Thruster handles HTTP compression and cache headers in production.
- Import Maps eliminates the need for JavaScript bundlers in most apps.
- Use Active Storage variants and lazy loading for image optimization.

## Code Examples

**Rails 8 asset pipeline — Import Maps for JavaScript, Propshaft for fingerprinting, Thruster for delivery**

```ruby
# config/importmap.rb — no bundler needed!
pin 'application', preload: true
pin '@hotwired/turbo-rails', to: 'turbo.min.js', preload: true
pin '@hotwired/stimulus', to: 'stimulus.min.js', preload: true
pin 'chartkick', to: 'chartkick.js'
pin 'Chart.bundle', to: 'Chart.bundle.js'

# Propshaft fingerprints all assets automatically
# application.js → application-a1b2c3d4.js
# Thruster serves them with compression and cache headers
```


## Resources

- [Rails Asset Pipeline (Propshaft)](https://guides.rubyonrails.org/asset_pipeline.html) — Official Rails guide on Propshaft asset pipeline

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*