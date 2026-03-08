---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-turbo-drive"
---

# Turbo Drive Basics

## Introduction
Turbo Drive intercepts every link click and form submission, fetches the response in the background, and swaps the page content without a full browser reload. The result is SPA-like speed with zero JavaScript from you.

## Key Concepts
- **Page Visit**: Turbo intercepts a link click and fetches the destination via `fetch`.
- **Body Swap**: Turbo replaces the `<body>` from the response, preserving `<head>` when possible.
- **Preview Cache**: Turbo caches page snapshots for instant back-button previews.
- **Progress Bar**: A thin bar shown when server responses take over 500ms.

## Real World Context
Without Turbo Drive, every link click causes the browser to tear down the page, request a new one, re-parse CSS/JS, and rebuild the DOM. Turbo Drive eliminates this overhead, saving 100-300ms per navigation in typical Rails apps.

## Deep Dive
### How Navigation Works

```
1. User clicks <a href="/products">
2. Turbo intercepts the click
3. Turbo sends fetch() to /products
4. Server renders full HTML as usual
5. Turbo extracts <body> from response
6. Turbo merges <head>, replaces <body>
7. Turbo updates URL and history
8. turbo:load event fires
```

The server doesn't know Turbo is involved. It renders a complete HTML page as always.

### It Works Automatically

All standard Rails links work with Turbo Drive:

```erb
<%= link_to "Products", products_path %>
<%= link_to "Edit", edit_product_path(@product) %>
<%= button_to "Delete", product_path(@product), method: :delete %>
```

No special attributes needed. Turbo intercepts all links within your domain by default.

### Opting Out

Use `data-turbo="false"` when you need a full page reload:

```erb
<%= link_to "Download PDF", report_path(format: :pdf),
    data: { turbo: false } %>

<div data-turbo="false">
  <!-- All links here bypass Turbo -->
</div>
```

The attribute is inherited by descendants.

### Important Events

```javascript
// Fires on every Turbo navigation (replaces DOMContentLoaded)
document.addEventListener("turbo:load", () => {
  // Initialize third-party libraries here
})

// Clean up before Turbo caches the page
document.addEventListener("turbo:before-cache", () => {
  // Remove tooltips, modals, temporary UI
})
```

`turbo:load` fires on every navigation, unlike `DOMContentLoaded` which fires only once. Use it for any initialization code, or better yet, use Stimulus controllers.

## Common Pitfalls
1. **Using DOMContentLoaded** — Only fires once. After Turbo navigations it won't fire again. Use `turbo:load` or Stimulus.
2. **Not handling preview cache** — Back-button shows cached snapshot. Open modals/tooltips appear in the preview. Clean up with `turbo:before-cache`.

## Best Practices
1. **Use Stimulus instead of event listeners** — Controllers auto-connect/disconnect with the DOM.
2. **Set cache control for dynamic pages** — Use `<meta name="turbo-cache-control" content="no-cache">` for frequently changing pages.

## Summary
- Turbo Drive intercepts links and forms, fetching pages via fetch and swapping `<body>`.
- Works automatically with all standard Rails links and forms.
- Use `data-turbo="false"` to opt out specific elements.
- `turbo:load` replaces `DOMContentLoaded` for code that runs on every page.
- Turbo caches pages for instant back-button previews.

## Code Examples

**A standard Rails controller works with Turbo Drive without any modifications**

```ruby
# Controllers need no changes for Turbo Drive
class ProductsController < ApplicationController
  def index
    @products = Product.all
    # Renders full HTML — Turbo extracts <body> client-side
  end

  def show
    @product = Product.find(params[:id])
    # Server has no idea Turbo is involved
  end
end
```


## Resources

- [Turbo Drive Handbook](https://turbo.hotwired.dev/handbook/drive) — Official handbook on Drive navigation and caching

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*