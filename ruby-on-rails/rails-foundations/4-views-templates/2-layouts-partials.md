---
source_course: "rails-foundations"
source_lesson: "rails-foundations-layouts-partials"
---

# Layouts and Partials

Layouts provide a consistent structure for your pages. Partials let you extract reusable view components.

## Layouts

The main layout wraps all your views:

```erb
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>
  <head>
    <title><%= content_for(:title) || "My App" %></title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>
    <%= stylesheet_link_tag "application" %>
    <%= javascript_importmap_tags %>
  </head>

  <body>
    <%= render "shared/navbar" %>

    <main class="container">
      <% if notice %>
        <div class="alert alert-success"><%= notice %></div>
      <% end %>
      <% if alert %>
        <div class="alert alert-danger"><%= alert %></div>
      <% end %>

      <%= yield %>
    </main>

    <%= render "shared/footer" %>
  </body>
</html>
```

### yield

The `yield` statement is where your view content gets inserted.

### content_for

Pass content from views to layouts:

```erb
<!-- In your view -->
<% content_for :title, "My Page Title" %>

<% content_for :sidebar do %>
  <div class="sidebar">
    <h3>Related Links</h3>
    ...
  </div>
<% end %>

<!-- In your layout -->
<title><%= content_for(:title) || "Default Title" %></title>
<aside><%= yield :sidebar %></aside>
```

## Partials

Partials are reusable view fragments. They start with an underscore:

### Basic Partial

```erb
<!-- app/views/articles/_article.html.erb -->
<article>
  <h2><%= article.title %></h2>
  <p><%= article.body %></p>
  <small>Posted <%= time_ago_in_words(article.created_at) %> ago</small>
</article>
```

Render it:

```erb
<!-- Explicit local variable -->
<%= render partial: "article", locals: { article: @article } %>

<!-- Shorthand (Rails infers the local variable name) -->
<%= render @article %>
```

### Rendering Collections

```erb
<!-- Renders _article.html.erb for each article -->
<%= render @articles %>

<!-- Equivalent to: -->
<% @articles.each do |article| %>
  <%= render article %>
<% end %>
```

### Partials with Spacers

```erb
<%= render partial: "article", collection: @articles, spacer_template: "article_divider" %>
```

### Shared Partials

```erb
<!-- app/views/shared/_navbar.html.erb -->
<nav>
  <%= link_to "Home", root_path %>
  <%= link_to "Articles", articles_path %>
</nav>

<!-- Render from any view -->
<%= render "shared/navbar" %>
```

### Partials with Local Variables

```erb
<!-- app/views/shared/_button.html.erb -->
<%= link_to text, path, class: "btn btn-#{style}" %>

<!-- Usage -->
<%= render "shared/button", text: "Edit", path: edit_article_path(@article), style: "primary" %>
```

## Resources

- [Layouts and Rendering in Rails](https://guides.rubyonrails.org/layouts_and_rendering.html) — Complete guide to layouts, templates, and partials

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*