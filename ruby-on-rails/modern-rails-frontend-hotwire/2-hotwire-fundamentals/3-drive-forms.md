---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-turbo-drive-forms"
---

# Forms with Turbo Drive

## Introduction
Turbo Drive handles form submissions the same way as links — intercepting, sending via fetch, and updating the page. However, forms require specific HTTP status codes to tell Turbo how to handle success and failure.

## Key Concepts
- **303 See Other**: Status for successful submissions that redirect. Turbo follows the redirect.
- **422 Unprocessable Entity**: Status for validation failures. Turbo re-renders the form with errors.
- **`turbo_submits_with`**: Data attribute that changes button text during submission.

## Real World Context
Every Rails app has forms. Turbo Drive makes all forms submit via fetch automatically. The key difference is returning correct HTTP status codes so Turbo knows whether to redirect or show errors.

## Deep Dive
### The Controller Pattern

```ruby
class ProductsController < ApplicationController
  def create
    @product = Product.new(product_params)
    if @product.save
      redirect_to @product, notice: "Product created."
    else
      render :new, status: :unprocessable_entity
    end
  end

  def update
    @product = Product.find(params[:id])
    if @product.update(product_params)
      redirect_to @product, notice: "Product updated."
    else
      render :edit, status: :unprocessable_entity
    end
  end
end
```

The key rule: **redirect on success, render with 422 on failure.** Without `status: :unprocessable_entity`, Turbo won't properly display validation errors.

### Submission Feedback

Prevent double submissions with `turbo_submits_with`:

```erb
<%= f.submit "Save Product",
    data: { turbo_submits_with: "Saving..." } %>
```

Turbo changes the text to "Saving..." and disables the button during submission. No JavaScript needed.

### Flash Messages

Flash messages work naturally since Turbo swaps the full body:

```erb
<body>
  <% flash.each do |type, message| %>
    <div class="flash flash-<%= type %>"><%= message %></div>
  <% end %>
  <%= yield %>
</body>
```

When Turbo follows a redirect, the destination page includes the flash in its HTML.

## Common Pitfalls
1. **Forgetting `status: :unprocessable_entity`** — Without 422, Turbo treats the form as a successful navigation and errors may not display.
2. **Double submissions** — Always add `turbo_submits_with` or disable the button during submission.

## Best Practices
1. **Follow redirect/render consistently** — `redirect_to` on success (303), `render` with 422 on failure.
2. **Add `turbo_submits_with` to all submit buttons** — Simple attribute, prevents double clicks.

## Summary
- Turbo Drive intercepts form submissions and sends them via fetch.
- Successful submissions redirect (303), failures render with 422.
- `data-turbo-submits-with` provides button feedback and prevents double clicks.
- Flash messages work naturally with body swapping.

## Code Examples

**Turbo-compatible controller: redirect on success, render 422 on failure**

```ruby
class CommentsController < ApplicationController
  def create
    @comment = @post.comments.build(comment_params)
    if @comment.save
      redirect_to @post, notice: "Comment added."
    else
      render :new, status: :unprocessable_entity
    end
  end
end
```


## Resources

- [Turbo Drive Forms](https://turbo.hotwired.dev/handbook/drive#form-submissions) — Official handbook on forms with Turbo Drive

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*