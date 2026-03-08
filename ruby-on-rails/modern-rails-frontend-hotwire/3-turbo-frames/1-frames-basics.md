---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-turbo-frames-basics"
---

# The Turbo Frame Tag

## Introduction
Turbo Frames let you decompose a page into independent sections that update without affecting the rest. Each frame is a `<turbo-frame>` tag with a unique ID. When a link or form inside a frame triggers a response, only the matching frame is extracted and swapped.

## Key Concepts
- **`turbo_frame_tag`**: Rails helper generating a `<turbo-frame>` element.
- **Frame Matching**: Turbo finds a `<turbo-frame>` with the same ID in the response and swaps only that content.
- **`dom_id`**: Rails helper generating unique IDs from models (e.g., `dom_id(@product)` → `"product_123"`).
- **Frame Scope**: Links/forms inside a frame only update that frame by default.

## Real World Context
Turbo Frames power inline editing, tabbed interfaces, search filters, comment sections, and modal-like overlays in production Rails apps. They replace complex JavaScript patterns for partial DOM updates.

## Deep Dive
### Basic Frame

```erb
<%= turbo_frame_tag dom_id(@product) do %>
  <p><%= @product.name %> — <%= number_to_currency(@product.price) %></p>
  <%= link_to "Edit", edit_product_path(@product) %>
<% end %>
```

This generates `<turbo-frame id="product_123">`. Clicking "Edit" fetches the edit page and swaps only the matching frame.

### Edit-in-Place Pattern

Both show and edit views use the same frame ID:

```erb
<!-- show.html.erb -->
<%= turbo_frame_tag dom_id(@product) do %>
  <p><%= @product.name %></p>
  <%= link_to "Edit", edit_product_path(@product) %>
<% end %>

<!-- edit.html.erb -->
<%= turbo_frame_tag dom_id(@product) do %>
  <%= form_with model: @product do |f| %>
    <%= f.text_field :name %>
    <%= f.submit "Save" %>
    <%= link_to "Cancel", product_path(@product) %>
  <% end %>
<% end %>
```

Clicking Edit swaps the display with the form inline. Saving redirects back, swapping the form back to display. No page reload.

### Breaking Out of Frames

Use `_top` for full-page navigation:

```erb
<%= turbo_frame_tag dom_id(@product) do %>
  <%= link_to "Edit", edit_product_path(@product) %>  <!-- frame only -->
  <%= link_to "Details", product_path(@product),
      data: { turbo_frame: "_top" } %>  <!-- full page -->
<% end %>
```

### Targeting Other Frames

Links can target any frame by ID:

```erb
<nav>
  <%= link_to "Electronics", products_path(category: "electronics"),
      data: { turbo_frame: "product_list" } %>
</nav>
<%= turbo_frame_tag "product_list" do %>
  <%= render @products %>
<% end %>
```

Clicking the nav link updates only the `product_list` frame. Powerful for filtering, tabs, and master-detail layouts.

## Common Pitfalls
1. **Mismatched frame IDs** — If the response doesn't contain a matching frame, Turbo shows an error.
2. **Forgetting `dom_id`** — Hardcoding IDs breaks with multiple items on the same page.

## Best Practices
1. **Use `dom_id` consistently** — Generates unique, predictable IDs like `product_123`.
2. **Keep frames self-contained** — A frame should contain its display content and the links/forms to modify it.

## Summary
- `turbo_frame_tag` wraps page sections for independent updates.
- Frame matching works by ID between source and response pages.
- Edit-in-place uses the same frame ID in show and edit views.
- `data-turbo-frame="_top"` breaks out for full-page navigation.
- Links can target any frame by specifying its ID.

## Code Examples

**Controllers work unchanged — Rails renders the full page and Turbo extracts the matching frame**

```ruby
# Controllers need no special handling for Turbo Frames
class ProductsController < ApplicationController
  def edit
    @product = Product.find(params[:id])
    # Renders full page — Turbo extracts only the matching frame
  end

  def update
    @product = Product.find(params[:id])
    if @product.update(product_params)
      redirect_to @product  # Turbo follows redirect, swaps frame
    else
      render :edit, status: :unprocessable_entity
    end
  end
end
```


## Resources

- [Turbo Frames Handbook](https://turbo.hotwired.dev/handbook/frames) — Official handbook covering frames, navigation, and targeting

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*