---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-advanced-frame-patterns"
---

# Advanced Frame Patterns

Combine frames for sophisticated UI patterns.

## Modal Pattern

```erb
<!-- Layout with modal container -->
<body>
  <%= yield %>
  
  <%= turbo_frame_tag 'modal' %>
</body>

<!-- Link that opens modal -->
<%= link_to 'Edit', edit_product_path(@product), 
    data: { turbo_frame: 'modal' } %>

<!-- products/edit.html.erb -->
<%= turbo_frame_tag 'modal' do %>
  <div class="modal-backdrop" data-action="click->modal#close">
    <div class="modal-content" data-controller="modal">
      <h2>Edit Product</h2>
      <%= form_with model: @product do |f| %>
        <%= f.text_field :name %>
        <%= f.submit 'Save' %>
      <% end %>
      
      <%= link_to 'Cancel', products_path, data: { turbo_frame: 'modal' } %>
    </div>
  </div>
<% end %>
```

## Pagination in Frames

```erb
<%= turbo_frame_tag 'products_list' do %>
  <div class="products">
    <%= render @products %>
  </div>
  
  <div class="pagination">
    <%= link_to 'Previous', products_path(page: @page - 1) if @page > 1 %>
    <%= link_to 'Next', products_path(page: @page + 1) if @products.any? %>
  </div>
<% end %>
```

## Tab Navigation

```erb
<div class="tabs">
  <nav>
    <%= link_to 'Details', product_details_path(@product), 
        data: { turbo_frame: 'tab_content' } %>
    <%= link_to 'Reviews', product_reviews_path(@product), 
        data: { turbo_frame: 'tab_content' } %>
    <%= link_to 'Related', product_related_path(@product), 
        data: { turbo_frame: 'tab_content' } %>
  </nav>
  
  <%= turbo_frame_tag 'tab_content', 
      src: product_details_path(@product) do %>
    <p>Loading...</p>
  <% end %>
</div>
```

## Frame Loading States

```css
turbo-frame {
  display: block;
}

turbo-frame[busy] {
  opacity: 0.5;
  pointer-events: none;
}

turbo-frame[busy]::after {
  content: '';
  position: absolute;
  /* Spinner styles */
}
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*