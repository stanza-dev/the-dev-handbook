---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-turbo-streams-basics"
---

# Turbo Stream Actions

## Introduction
Turbo Streams let you perform targeted DOM manipulations from the server. Instead of replacing entire pages or frames, you send small HTML fragments with instructions like "append this to the list" or "replace this element." Streams work in HTTP responses and over WebSockets.

## Key Concepts
- **Stream Action**: An instruction like `append`, `prepend`, `replace`, `update`, `remove`, or `morph` that tells Turbo what to do with an HTML fragment.
- **Target**: The DOM element ID that the action operates on.
- **`turbo_stream` Helper**: Rails helper for generating stream responses in controllers.
- **Morph Action**: New in Turbo 8 — surgically updates an element by diffing its current content against new content, preserving DOM state.

## Real World Context
Turbo Streams power features like adding a new comment to a list without reloading, removing a deleted item from a table, showing flash notifications, and updating counters. They work in both synchronous HTTP responses and asynchronous WebSocket broadcasts.

## Deep Dive
### Available Actions

Turbo provides seven built-in stream actions:

```ruby
# append — add to the end of a container
turbo_stream.append "messages", partial: "messages/message", locals: { message: @message }

# prepend — add to the beginning
turbo_stream.prepend "messages", partial: "messages/message", locals: { message: @message }

# replace — replace the entire element (including the element itself)
turbo_stream.replace dom_id(@message), partial: "messages/message", locals: { message: @message }

# update — replace only the inner content
turbo_stream.update dom_id(@message), partial: "messages/message", locals: { message: @message }

# remove — remove the element from the DOM
turbo_stream.remove dom_id(@message)

# before / after — insert adjacent to the target
turbo_stream.before dom_id(@message), partial: "messages/new_message"
turbo_stream.after dom_id(@message), partial: "messages/reply"
```

Each action targets a DOM element by ID and performs the specified operation.

### Morph Action (Turbo 8)

The `morph` action is a significant addition in Turbo 8. Instead of replacing HTML wholesale, it diffs the current and new content, updating only what changed:

```ruby
# morph — intelligent diff-based update preserving DOM state
turbo_stream.morph dom_id(@product), partial: "products/product", locals: { product: @product }
```

Morph preserves form input values, scroll positions, CSS transitions, and focus state that a full `replace` would destroy. This is ideal for live-updating dashboards and collaborative editing.

### Stream Responses from Controllers

Return streams from a controller action by responding to `turbo_stream` format:

```ruby
class MessagesController < ApplicationController
  def create
    @message = @chat.messages.create!(message_params)

    respond_to do |format|
      format.turbo_stream  # renders create.turbo_stream.erb
      format.html { redirect_to @chat }
    end
  end

  def destroy
    @message = Message.find(params[:id])
    @message.destroy

    respond_to do |format|
      format.turbo_stream { render turbo_stream: turbo_stream.remove(dom_id(@message)) }
      format.html { redirect_to @chat }
    end
  end
end
```

The `format.html` fallback ensures the app works without JavaScript. The stream template for create:

```erb
<!-- messages/create.turbo_stream.erb -->
<%= turbo_stream.append "messages", @message %>
<%= turbo_stream.update "message_count", html: "#{@chat.messages.count} messages" %>
<%= turbo_stream.replace "new_message_form", partial: "messages/form", locals: { message: Message.new } %>
```

A single response can contain multiple stream actions — append the message, update the counter, and reset the form.

## Common Pitfalls
1. **Forgetting the target element ID** — Stream actions target elements by ID. If the target doesn't exist in the DOM, the action silently fails.
2. **Using replace when morph is better** — Replace destroys DOM state (focus, scroll position, open dropdowns). Use morph when you need to preserve state.

## Best Practices
1. **Include HTML fallback** — Always handle `format.html` in addition to `format.turbo_stream` for progressive enhancement.
2. **Use morph for live updates** — When updating existing content (dashboards, collaborative editing), morph provides a smoother experience.

## Summary
- Turbo Streams perform targeted DOM operations: append, prepend, replace, update, remove, before, after, and morph.
- Controllers respond to `turbo_stream` format with stream templates.
- Multiple actions can be combined in a single response.
- Morph (Turbo 8) diffs content intelligently, preserving DOM state.
- Always include HTML fallback for progressive enhancement.

## Code Examples

**A stream template combining append, update, and replace — adds a message, updates the count, and resets the form**

```ruby
# messages/create.turbo_stream.erb
# Multiple stream actions in a single response:
<%= turbo_stream.append "messages", @message %>
<%= turbo_stream.update "message_count",
    html: "#{@chat.messages.count} messages" %>
<%= turbo_stream.replace "new_message_form",
    partial: "messages/form",
    locals: { message: Message.new } %>
```


## Resources

- [Turbo Streams Handbook](https://turbo.hotwired.dev/handbook/streams) — Official handbook on Turbo Stream actions and responses

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*