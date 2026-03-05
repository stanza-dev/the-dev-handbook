---
source_course: "rails-foundations"
source_lesson: "rails-foundations-erb-templates"
---

# ERB Templates and Syntax

## Introduction

ERB (Embedded Ruby) is Rails' default template engine. It lets you embed Ruby code directly in HTML, turning static pages into dynamic, data-driven views.

## Key Concepts

- **Output tag `<%= %>`**: Evaluates the Ruby expression and inserts the result into the HTML output.
- **Execution tag `<% %>`**: Runs Ruby code (loops, conditionals) without outputting anything.
- **Comment tag `<%# %>`**: An ERB comment that is not rendered in the HTML output.
- **Auto-escaping**: Rails automatically escapes HTML in output tags to prevent XSS attacks.

## Real World Context

Every Rails view is an ERB template by default. Understanding the difference between `<%= %>` (output) and `<% %>` (execute) is fundamental to building any Rails interface. ERB's auto-escaping protects your application from cross-site scripting attacks without extra effort.

## Deep Dive

### Output vs Execution Tags

```erb
<!-- Output: inserts the value into HTML -->
<h1><%= @article.title %></h1>

<!-- Execution: runs code but doesn't output -->
<% if @article.published? %>
  <span class="badge">Published</span>
<% else %>
  <span class="badge">Draft</span>
<% end %>
```

### Loops

```erb
<ul>
<% @articles.each do |article| %>
  <li><%= article.title %></li>
<% end %>
</ul>
```

### View File Locations

```
app/views/
  articles/
    index.html.erb     # ArticlesController#index
    show.html.erb      # ArticlesController#show
    _article.html.erb  # Partial (note the underscore)
  layouts/
    application.html.erb  # Main layout
```

### Auto-Escaping (XSS Protection)

```erb
<%= "<script>alert('xss')</script>" %>
<!-- Outputs: &lt;script&gt;alert('xss')&lt;/script&gt; -->

<!-- To output raw HTML (use with caution): -->
<%= raw @article.html_content %>
```

### Instance Variables from Controllers

```ruby
# Controller
def show
  @article = Article.find(params[:id])
  @comments = @article.comments.recent
end
```

```erb
<!-- View has access to @article and @comments -->
<h1><%= @article.title %></h1>
<% @comments.each do |comment| %>
  <p><%= comment.body %></p>
<% end %>
```

## Common Pitfalls

- **Using `<%= %>` for control flow**: Conditionals and loops should use `<% %>` (no equals sign). Using `<%= if ... %>` outputs the return value of the conditional.
- **Forgetting to close ERB tags**: Every `<% if %>` needs a matching `<% end %>`.
- **Using `html_safe` carelessly**: Marking user input as `html_safe` bypasses XSS protection and creates security vulnerabilities.

## Best Practices

- Use `<%= %>` only when you want to display a value in the HTML.
- Never mark user-provided content as `html_safe`.
- Keep complex logic out of views; use helper methods or presenters instead.

## Summary

- ERB embeds Ruby in HTML using `<%= %>` (output) and `<% %>` (execution) tags.
- Views are stored in `app/views/controller_name/` and matched to actions automatically.
- Rails auto-escapes HTML output to prevent XSS attacks.
- Instance variables set in controllers are available in the corresponding view.
- Use `<%# %>` for comments that should not appear in the rendered HTML.

## Code Examples

**ERB uses two main tags: <%= %> outputs values to HTML, while <% %> executes Ruby code silently (for loops, conditionals, etc.).**

```ruby
# ERB output tag - displays the value
# <%= @article.title %>

# ERB execution tag - runs code without output
# <% if @article.published? %>
#   <span>Published</span>
# <% end %>

# Looping over a collection
# <% @articles.each do |article| %>
#   <li><%= article.title %></li>
# <% end %>
```


## Resources

- [Action View Overview](https://guides.rubyonrails.org/action_view_overview.html) — Complete guide to Rails views

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*