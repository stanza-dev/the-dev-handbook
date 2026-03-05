---
source_course: "rails-foundations"
source_lesson: "rails-foundations-layouts-partials"
---

# Layouts and Partials

## Introduction

Layouts provide a consistent wrapper for every page in your application, while partials let you extract reusable view fragments. Together they keep your views DRY and maintainable.

## Key Concepts

- **Layout**: A template that wraps all views, providing shared HTML structure (head, navbar, footer). Located at `app/views/layouts/application.html.erb`.
- **`yield`**: The point in the layout where the current view's content is inserted.
- **Partial**: A reusable view fragment whose filename starts with an underscore (e.g., `_article.html.erb`).
- **`content_for`**: A mechanism to pass content from a view into named sections of the layout.

## Real World Context

Every web application needs consistent navigation, footer, and metadata across pages. Layouts handle this. When you display the same article card on the index page and the homepage, a partial prevents code duplication. Large Rails apps can have hundreds of partials.

## Deep Dive

### The Application Layout

```erb
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>
  <head>
    <title><%= content_for(:title) || "My App" %></title>
    <%= csrf_meta_tags %>
    <%= stylesheet_link_tag "application" %>
  </head>
  <body>
    <%= render "shared/navbar" %>
    <main>
      <% if notice %>
        <div class="alert"><%= notice %></div>
      <% end %>
      <%= yield %>
    </main>
    <%= render "shared/footer" %>
  </body>
</html>
```

### Partials

```erb
<!-- app/views/articles/_article.html.erb -->
<article>
  <h2><%= article.title %></h2>
  <p><%= article.body %></p>
</article>

<!-- Render a single partial -->
<%= render partial: "article", locals: { article: @article } %>

<!-- Shorthand -->
<%= render @article %>

<!-- Render a collection -->
<%= render @articles %>
```

### content_for

```erb
<!-- In your view -->
<% content_for :title, "My Page Title" %>

<!-- In your layout -->
<title><%= content_for(:title) || "Default Title" %></title>
```

### Partials with Local Variables

```erb
<!-- app/views/shared/_button.html.erb -->
<%= link_to text, path, class: "btn btn-#{style}" %>

<!-- Usage -->
<%= render "shared/button", text: "Edit", path: edit_article_path(@article), style: "primary" %>
```

## Common Pitfalls

- **Including the underscore when rendering**: Write `render "article"`, not `render "_article"`. Rails adds the underscore automatically.
- **Too many partials**: Extracting every line into a partial hurts readability. Extract when you have genuine reuse or a partial simplifies a complex view.
- **Forgetting local variables**: Partials do not have access to instance variables by convention. Pass data explicitly via `locals:` or the shorthand.

## Best Practices

- Use the application layout for elements shared across all pages (navigation, footer, flash messages).
- Extract partials when the same markup appears in multiple views.
- Use `render @collection` for rendering lists, which is faster than manual loops.

## Summary

- Layouts wrap views with shared HTML structure; `yield` inserts the view content.
- Partials are reusable view fragments with filenames starting with `_`.
- Render partials with `render "name"` (without the underscore prefix).
- `render @articles` efficiently renders a partial for each item in the collection.
- Use `content_for` to pass content from views to specific layout sections.

## Code Examples

**Partials are rendered without the underscore prefix. Collections can be rendered in one call, and layouts use yield to insert view content.**

```ruby
# Rendering a partial
# <%= render "article", article: @article %>

# Rendering a collection (renders _article.html.erb for each)
# <%= render @articles %>

# Layout with yield
# <body>
#   <%= render "shared/navbar" %>
#   <main><%= yield %></main>
#   <%= render "shared/footer" %>
# </body>
```


## Resources

- [Layouts and Rendering in Rails](https://guides.rubyonrails.org/layouts_and_rendering.html) — Complete guide to layouts, templates, and partials

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*