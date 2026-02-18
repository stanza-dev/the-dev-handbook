---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-asset-optimization"
---

# Asset Optimization

Frontend assets significantly impact page load times.

## Asset Pipeline Configuration

```ruby
# config/environments/production.rb

# Enable compression
config.assets.css_compressor = :sass
config.assets.js_compressor = :terser

# Enable asset fingerprinting for cache busting
config.assets.digest = true

# Compile assets
config.assets.compile = false  # Precompile in deployment
```

## Image Optimization

```ruby
# Use Active Storage variants
class Product < ApplicationRecord
  has_one_attached :image
  
  def thumbnail
    image.variant(resize_to_limit: [200, 200]).processed
  end
  
  def optimized_image
    image.variant(
      resize_to_limit: [800, 600],
      saver: { quality: 80 }  # JPEG quality
    ).processed
  end
end
```

```erb
<!-- Responsive images -->
<%= image_tag @product.image.variant(resize_to_limit: [400, 300]),
    srcset: {
      "#{url_for(@product.image.variant(resize_to_limit: [400, 300]))} 1x",
      "#{url_for(@product.image.variant(resize_to_limit: [800, 600]))} 2x"
    },
    loading: 'lazy' %>
```

## Import Maps (Rails 7+)

```ruby
# config/importmap.rb
pin 'application', preload: true
pin '@hotwired/turbo-rails', to: 'turbo.min.js', preload: true
pin '@hotwired/stimulus', to: 'stimulus.min.js', preload: true
```

## Critical CSS

Inline critical CSS for above-the-fold content:

```erb
<head>
  <style>
    <%= Rails.application.assets['critical.css'].to_s.html_safe %>
  </style>
  <%= stylesheet_link_tag 'application', media: 'print', onload: "this.media='all'" %>
</head>
```

## Preloading Important Assets

```erb
<%= preload_link_tag 'application.css' %>
<%= preload_link_tag 'application.js' %>
<%= preload_link_tag image_path('logo.png'), as: 'image' %>
```

## CDN Configuration

```ruby
# config/environments/production.rb
config.asset_host = 'https://cdn.example.com'
config.action_controller.asset_host = 'https://cdn.example.com'
```

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*