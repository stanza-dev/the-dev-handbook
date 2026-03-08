---
source_course: "rails-security-hardening"
source_lesson: "rails-security-sanitizing-html"
---

# Sanitizing HTML

## Introduction
Sometimes you need to allow some HTML in user content — rich text editors, markdown rendering, or formatted comments. The `sanitize` helper lets you allow safe tags while stripping dangerous ones like `<script>`.

## Key Concepts
- **Sanitize Helper**: A Rails view helper that strips dangerous HTML tags while allowing a configurable set of safe tags.
- **Allowlist**: A list of explicitly permitted HTML tags and attributes. Everything not on the list is stripped.
- **Tag Stripping**: Removing HTML tags entirely while preserving the text content inside them.

## Real World Context
Blog platforms, forums, and any application with rich text editing need HTML sanitization. Without it, you must choose between no formatting and full XSS vulnerability. Sanitization gives you a safe middle ground.

## Deep Dive

### The sanitize Helper

Use `sanitize` in views to allow specific HTML tags:

```erb
<%= sanitize @article.body %>
```

By default, `sanitize` allows basic formatting tags (strong, em, p, etc.) and removes dangerous ones like `<script>`, `<iframe>`, and event handlers.

### Custom Allowed Tags

You can specify exactly which tags to allow:

```erb
<%= sanitize @article.body, tags: %w[p br strong em a ul ol li] %>
```

This permits only paragraph, line break, bold, italic, link, and list tags. Everything else is stripped.

### Allowing Attributes

Control which attributes are permitted on allowed tags:

```erb
<%= sanitize @article.body,
    tags: %w[p a img],
    attributes: %w[href src alt] %>
```

This allows links with `href`, images with `src` and `alt`, but strips `onclick`, `onerror`, and other dangerous attributes.

### Application-Wide Configuration

Set default allowed tags in an initializer to avoid repeating the list:

```ruby
# config/initializers/sanitize.rb
Rails.application.config.action_view.sanitized_allowed_tags = %w[
  p br strong em a ul ol li h2 h3 blockquote
]

Rails.application.config.action_view.sanitized_allowed_attributes = %w[
  href title
]
```

This configuration applies to all `sanitize` calls that do not specify their own tag list.

### Sanitizing in Models

You can sanitize content before saving to the database using a callback:

```ruby
class Comment < ApplicationRecord
  before_save :sanitize_content

  private

  def sanitize_content
    self.body = ActionController::Base.helpers.sanitize(
      body,
      tags: %w[p br strong em]
    )
  end
end
```

Sanitizing on save ensures that dangerous HTML never persists in your database.

## Common Pitfalls
1. **Allowing `style` attributes** — Inline styles can be used for CSS-based attacks like data exfiltration. Only allow `style` if absolutely necessary.
2. **Forgetting `javascript:` URLs** — Even if you allow `<a>` tags, an attacker can use `<a href="javascript:alert('XSS')">` to execute scripts.

## Best Practices
1. **Sanitize on input and output** — Sanitize when saving to the database AND when rendering. Belt and suspenders.
2. **Use the strictest allowlist possible** — Start with no tags allowed and add only what your feature requires.

## Summary
- The `sanitize` helper strips dangerous HTML while allowing safe formatting tags.
- Configure allowed tags and attributes explicitly.
- Set application-wide defaults in an initializer.
- Sanitize both on input (before_save) and output (in views) for maximum safety.

## Code Examples

**Two-layer sanitization strategy — sanitize in the model before saving and in the view when rendering**

```ruby
# View: allow only formatting tags
# <%= sanitize @article.body, tags: %w[p br strong em a], attributes: %w[href] %>

# Model: sanitize before saving
class Comment < ApplicationRecord
  before_save :sanitize_content

  private

  def sanitize_content
    self.body = ActionController::Base.helpers.sanitize(
      body, tags: %w[p br strong em]
    )
  end
end
```


## Resources

- [ActionView::Helpers::SanitizeHelper](https://api.rubyonrails.org/classes/ActionView/Helpers/SanitizeHelper.html) — API reference for Rails sanitize helpers

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*