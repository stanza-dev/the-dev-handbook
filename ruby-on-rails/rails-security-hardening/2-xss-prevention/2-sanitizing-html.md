---
source_course: "rails-security-hardening"
source_lesson: "rails-security-sanitizing-html"
---

# Sanitizing HTML

Sometimes you need to allow some HTML (like in a rich text editor) while blocking dangerous tags.

## Rails Sanitize Helper

Use `sanitize` to allow specific HTML tags:

```erb
<%= sanitize @article.body %>
```

By default, sanitize allows basic formatting tags and removes dangerous ones like `<script>`.

## Custom Allowed Tags

```erb
<%= sanitize @article.body, tags: %w[p br strong em a ul ol li] %>
```

## Allowing Attributes

```erb
<%= sanitize @article.body,
    tags: %w[p a img],
    attributes: %w[href src alt] %>
```

## Application-Wide Configuration

Set defaults in an initializer:

```ruby
# config/initializers/sanitize.rb
Rails.application.config.action_view.sanitized_allowed_tags = %w[
  p br strong em a ul ol li h2 h3 blockquote
]

Rails.application.config.action_view.sanitized_allowed_attributes = %w[
  href title
]
```

## Sanitizing in Models

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

## Link Safety

Prevent javascript: URLs in links:

```erb
<!-- Safe - Rails sanitizes href -->
<%= link_to 'Click', @url %>

<!-- Manual URL validation -->
<% if @url.start_with?('http://', 'https://') %>
  <%= link_to 'Visit', @url %>
<% end %>
```

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*