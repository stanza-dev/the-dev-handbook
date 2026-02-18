---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-turbo-frames-basics"
---

# Turbo Frames Basics

Turbo Frames let you update portions of a page independently.

## Creating a Frame

```erb
<%= turbo_frame_tag 'product_info' do %>
  <h2><%= @product.name %></h2>
  <p><%= @product.description %></p>
  <%= link_to 'Edit', edit_product_path(@product) %>
<% end %>
```

## How Frames Work

When you click a link inside a frame:
1. Turbo fetches the target page
2. Finds a matching frame (same ID)
3. Extracts just that frame's content
4. Swaps it into the current page

## Frame IDs with dom_id

```erb
<!-- Using Rails dom_id helper -->
<%= turbo_frame_tag dom_id(@product) do %>
  <%= render @product %>
<% end %>

<!-- Generates: <turbo-frame id="product_123">...</turbo-frame> -->
```

## Edit-in-Place Pattern

```erb
<!-- products/show.html.erb -->
<%= turbo_frame_tag dom_id(@product) do %>
  <h2><%= @product.name %></h2>
  <%= link_to 'Edit', edit_product_path(@product) %>
<% end %>

<!-- products/edit.html.erb -->
<%= turbo_frame_tag dom_id(@product) do %>
  <%= form_with model: @product do |f| %>
    <%= f.text_field :name %>
    <%= f.submit 'Save' %>
    <%= link_to 'Cancel', @product %>
  <% end %>
<% end %>
```

Clicking "Edit" replaces the display with the form. Submitting returns to show.

## Breaking Out of Frames

```erb
<%= turbo_frame_tag 'sidebar' do %>
  <!-- This link navigates the entire page, not just the frame -->
  <%= link_to 'View All', products_path, data: { turbo_frame: '_top' } %>
<% end %>
```

## Frame Targets

```erb
<!-- Target a different frame -->
<%= link_to 'Details', product_path(@product), 
    data: { turbo_frame: 'main_content' } %>
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*