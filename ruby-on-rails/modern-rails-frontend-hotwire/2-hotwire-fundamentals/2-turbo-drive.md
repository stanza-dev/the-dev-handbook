---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-turbo-drive"
---

# Turbo Drive

Turbo Drive makes navigation instant by fetching pages in the background and swapping content.

## How It Works

Turbo Drive automatically:
1. Intercepts link clicks and form submissions
2. Fetches the page via fetch API
3. Extracts the `<body>` content
4. Swaps it into the current page
5. Updates the URL and history

## It Just Works

```erb
<!-- These work automatically with Turbo Drive -->
<%= link_to 'Products', products_path %>

<%= form_with model: @product do |f| %>
  <%= f.text_field :name %>
  <%= f.submit 'Save' %>
<% end %>
```

## Disabling Turbo Drive

```erb
<!-- For specific link -->
<%= link_to 'Download', file_path, data: { turbo: false } %>

<!-- For an entire section -->
<div data-turbo="false">
  <%= link_to 'External', 'https://example.com' %>
</div>

<!-- Disable for form -->
<%= form_with url: upload_path, data: { turbo: false } do |f| %>
  <%= f.file_field :document %>
  <%= f.submit 'Upload' %>
<% end %>
```

## Progress Bar

Turbo shows a progress bar for slow requests:

```css
.turbo-progress-bar {
  height: 4px;
  background-color: #3b82f6;
}
```

## Turbo Drive Events

```javascript
// Before navigation starts
document.addEventListener('turbo:before-visit', (event) => {
  console.log('Navigating to:', event.detail.url)
})

// After page loads
document.addEventListener('turbo:load', () => {
  console.log('Page loaded')
  // Initialize third-party scripts here
})

// Page is about to be cached
document.addEventListener('turbo:before-cache', () => {
  // Clean up before caching
})
```

## Cache Control

```erb
<!-- Skip cache for this page -->
<head>
  <meta name="turbo-cache-control" content="no-cache">
</head>
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*