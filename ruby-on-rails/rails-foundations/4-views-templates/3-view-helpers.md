---
source_course: "rails-foundations"
source_lesson: "rails-foundations-view-helpers"
---

# View Helpers and Asset Tags

## Introduction

Rails provides dozens of built-in helper methods that generate HTML for you. Instead of writing raw HTML tags for links, images, and forms, you use helpers that handle URL generation, asset fingerprinting, and security automatically.

## Key Concepts

- **`link_to`**: Generates `<a>` tags with proper URLs from route helpers.
- **`image_tag`**: Generates `<img>` tags with asset pipeline integration.
- **`content_for` / `yield`**: A mechanism for views to inject content into specific regions of the layout.
- **Custom helpers**: Methods defined in `app/helpers/` that are available in all views.

## Real World Context

Using helpers instead of raw HTML is not just a convenience — it is a security practice. `link_to` automatically escapes content to prevent XSS. Asset tag helpers add fingerprints for cache busting. Form helpers include CSRF tokens. Skipping helpers means skipping Rails' built-in protections.

## Deep Dive

### Link Helpers

The `link_to` helper generates anchor tags:

```erb
<%# Basic link %>
<%= link_to "Home", root_path %>
<%# Generates: <a href="/">Home</a> %>

<%# Link with CSS class %>
<%= link_to "Edit", edit_article_path(@article), class: "btn btn-primary" %>
```

The first argument is the link text, the second is the URL (usually a route helper), and optional keyword arguments become HTML attributes.

### Image and Asset Tags

```erb
<%# Image from app/assets/images/ %>
<%= image_tag "logo.png", alt: "Company Logo", width: 200 %>
<%# Generates: <img src="/assets/logo-abc123.png" alt="Company Logo" width="200" /> %>
```

Notice the fingerprint (`abc123`) in the src. This ensures browsers fetch the latest version when the file changes.

### content_for and yield

Layouts can define named regions that views fill in:

```erb
<%# app/views/layouts/application.html.erb %>
<html>
<head>
  <title><%= content_for?(:title) ? yield(:title) : "My App" %></title>
</head>
<body>
  <%= yield %>  <%# Main content goes here %>
</body>
</html>

<%# app/views/articles/show.html.erb %>
<% content_for :title, @article.title %>
<h1><%= @article.title %></h1>
```

The view uses `content_for` to inject content into named regions. The layout uses `yield(:name)` to render them.

### Custom Helpers

```ruby
# app/helpers/articles_helper.rb
module ArticlesHelper
  def reading_time(article)
    minutes = (article.body.split.size / 200.0).ceil
    "#{minutes} min read"
  end
end
```

Helpers defined in `app/helpers/` are automatically available in views. Use them to keep templates clean.

## Common Pitfalls

- **Using raw HTML for links**: Writing `<a href="/articles/1">` bypasses route helpers and XSS protection.
- **Forgetting `alt` on images**: Always pass `alt:` to `image_tag` for accessibility.
- **Putting logic in views**: If a helper grows complex, consider a presenter or decorator pattern.

## Best Practices

- Always use `link_to` and route helpers instead of hardcoded URLs.
- Use `content_for` / `yield` for page-specific titles, meta tags, and sidebar content.
- Keep custom helpers focused on presentation — business logic belongs in models or services.

## Summary

- `link_to` generates safe, route-aware anchor tags with automatic XSS escaping.
- `image_tag` and other asset helpers add cache-busting fingerprints.
- `content_for` / `yield` let views inject content into specific layout regions.
- Custom helpers in `app/helpers/` keep view templates clean and DRY.
- Always prefer Rails helpers over raw HTML for security and maintainability.

## Code Examples

**Common view helpers — link_to for navigation, image_tag for assets, and custom helpers for reusable presentation logic.**

```ruby
# View helpers
<%= link_to "Home", root_path %>
<%= link_to "Edit", edit_article_path(@article), class: "btn" %>
<%= image_tag "logo.png", alt: "Logo", width: 200 %>

# Custom helper in app/helpers/application_helper.rb
module ApplicationHelper
  def page_title(title = nil)
    title ? "#{title} | MyApp" : "MyApp"
  end
end
```


## Resources

- [Action View Helpers](https://guides.rubyonrails.org/action_view_helpers.html) — Official guide to all built-in view helpers in Rails

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*