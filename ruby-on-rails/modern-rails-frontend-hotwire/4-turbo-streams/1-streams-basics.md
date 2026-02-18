---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-turbo-streams-basics"
---

# Turbo Streams Basics

Turbo Streams can update, append, prepend, remove, or replace elements anywhere on the page.

## Stream Actions

| Action | Description |
|--------|-------------|
| append | Add to end of container |
| prepend | Add to start of container |
| replace | Replace entire element |
| update | Replace element's content |
| remove | Remove the element |
| before | Insert before element |
| after | Insert after element |

## Responding with Streams

```ruby
class CommentsController < ApplicationController
  def create
    @comment = @post.comments.create!(comment_params)
    
    respond_to do |format|
      format.turbo_stream
      format.html { redirect_to @post }
    end
  end
end
```

```erb
<!-- app/views/comments/create.turbo_stream.erb -->
<%= turbo_stream.append 'comments', @comment %>
<%= turbo_stream.update 'comment_count', @post.comments.count %>
<%= turbo_stream.replace 'comment_form', partial: 'comments/form', locals: { comment: Comment.new } %>
```

## Stream Tag Helpers

```erb
<!-- Append partial to container -->
<%= turbo_stream.append 'comments', partial: 'comment', locals: { comment: @comment } %>

<!-- Replace with rendered content -->
<%= turbo_stream.replace dom_id(@product) do %>
  <%= render @product %>
<% end %>

<!-- Remove element -->
<%= turbo_stream.remove dom_id(@comment) %>

<!-- Update just inner content -->
<%= turbo_stream.update 'cart_count', '5' %>
```

## Multiple Updates

```erb
<!-- comments/destroy.turbo_stream.erb -->
<%= turbo_stream.remove dom_id(@comment) %>
<%= turbo_stream.update 'comment_count', @post.comments.count %>
<%= turbo_stream.prepend 'flash', partial: 'shared/flash', locals: { message: 'Comment deleted' } %>
```

## Inline Stream Response

```ruby
def create
  @comment = @post.comments.create!(comment_params)
  
  respond_to do |format|
    format.turbo_stream {
      render turbo_stream: [
        turbo_stream.append('comments', @comment),
        turbo_stream.update('comment_count', @post.comments.count)
      ]
    }
  end
end
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*