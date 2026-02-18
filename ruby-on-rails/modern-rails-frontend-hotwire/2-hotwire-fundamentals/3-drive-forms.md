---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-turbo-drive-forms"
---

# Turbo Drive Forms

Turbo Drive handles forms automatically but requires proper response codes.

## Form Submission Flow

```ruby
class ProductsController < ApplicationController
  def create
    @product = Product.new(product_params)
    
    if @product.save
      # 303 redirect - Turbo follows it
      redirect_to @product, notice: 'Product created!'
    else
      # 422 status - Turbo replaces page content
      render :new, status: :unprocessable_entity
    end
  end
end
```

## Response Status Codes

| Status | Turbo Behavior |
|--------|----------------|
| 200 | Replaces page content |
| 303 | Follows redirect |
| 422 | Replaces page (validation errors) |
| 500 | Replaces page (error page) |

## Flash Messages

```erb
<!-- app/views/layouts/application.html.erb -->
<body>
  <div id="flash">
    <% flash.each do |type, message| %>
      <div class="flash flash-<%= type %>"><%= message %></div>
    <% end %>
  </div>
  
  <%= yield %>
</body>
```

## Form Submission States

```erb
<%= form_with model: @product do |f| %>
  <%= f.submit 'Save', 
      data: { 
        turbo_submits_with: 'Saving...',
        disable_with: 'Saving...'
      } %>
<% end %>
```

## Handling File Uploads

File uploads need special handling:

```erb
<!-- Direct uploads with Active Storage -->
<%= form_with model: @product, data: { turbo: false } do |f| %>
  <%= f.file_field :image, direct_upload: true %>
  <%= f.submit 'Upload' %>
<% end %>
```

Or use Turbo with proper progress:

```javascript
import { DirectUpload } from '@rails/activestorage'

// Handle upload progress
addEventListener('direct-upload:progress', (event) => {
  const { progress } = event.detail
  // Update progress UI
})
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*