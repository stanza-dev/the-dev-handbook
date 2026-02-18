---
source_course: "rails-foundations"
source_lesson: "rails-foundations-erb-templates"
---

# ERB Templates and Syntax

ERB (Embedded Ruby) lets you embed Ruby code in HTML templates. It's the default template engine in Rails.

## ERB Tags

Two types of ERB tags:

### Output Tags `<%= %>`

Outputs the result to HTML:

```erb
<h1><%= @article.title %></h1>
<p>Published: <%= @article.created_at.strftime("%B %d, %Y") %></p>
<p>Author: <%= @article.author.name %></p>
```

### Execution Tags `<% %>`

Executes Ruby but doesn't output:

```erb
<% if @article.published? %>
  <span class="badge">Published</span>
<% else %>
  <span class="badge">Draft</span>
<% end %>

<ul>
<% @articles.each do |article| %>
  <li><%= article.title %></li>
<% end %>
</ul>
```

## View File Locations

Views live in `app/views/` organized by controller:

```
app/views/
├── articles/
│   ├── index.html.erb    # ArticlesController#index
│   ├── show.html.erb     # ArticlesController#show
│   ├── new.html.erb      # ArticlesController#new
│   ├── edit.html.erb     # ArticlesController#edit
│   └── _article.html.erb # Partial (note the underscore)
├── layouts/
│   └── application.html.erb  # Main layout
└── shared/
    └── _navbar.html.erb  # Shared partial
```

## Common ERB Patterns

### Conditionals

```erb
<% if current_user %>
  <p>Welcome, <%= current_user.name %>!</p>
<% else %>
  <%= link_to "Log in", login_path %>
<% end %>
```

### Loops

```erb
<ul>
  <% @products.each do |product| %>
    <li>
      <%= product.name %> - <%= number_to_currency(product.price) %>
    </li>
  <% end %>
</ul>
```

### Safe HTML Output

By default, Rails escapes HTML to prevent XSS:

```erb
<%= "<script>alert('xss')</script>" %>
<!-- Outputs: &lt;script&gt;alert('xss')&lt;/script&gt; -->

<!-- To output raw HTML (be careful!): -->
<%= raw @article.html_content %>
<!-- Or -->
<%= @article.html_content.html_safe %>
```

### Comments

```erb
<%# This is an ERB comment - not rendered in HTML %>

<!-- This is an HTML comment - visible in source -->
```

## Instance Variables

Controllers pass data to views via instance variables:

```ruby
# Controller
def show
  @article = Article.find(params[:id])
  @comments = @article.comments.recent
end
```

```erb
<!-- View -->
<h1><%= @article.title %></h1>

<h2>Comments</h2>
<% @comments.each do |comment| %>
  <p><%= comment.body %></p>
<% end %>
```

## Resources

- [Action View Overview](https://guides.rubyonrails.org/action_view_overview.html) — Complete guide to Rails views

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*