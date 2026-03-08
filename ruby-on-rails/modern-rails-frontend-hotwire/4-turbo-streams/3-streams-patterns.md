---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-streams-patterns"
---

# Turbo Stream Responses

## Introduction
Beyond broadcasting, Turbo Streams can be returned directly from controller actions as HTTP responses. When a form submission or link request accepts the `turbo_stream` format, the controller can respond with targeted DOM updates instead of a full page redirect.

## Key Concepts
- **`format.turbo_stream`**: A respond_to block format for returning stream responses.
- **Stream Template**: An `.turbo_stream.erb` file containing stream actions.
- **Inline Streams**: Stream actions rendered directly in the controller without a template file.
- **Multiple Actions**: A single response can contain several stream actions.

## Real World Context
Stream responses are ideal for form submissions that need to update multiple parts of the page: adding an item to a list, updating a counter, resetting the form, and showing a flash message — all in one response without a page reload.

## Deep Dive
### Template-Based Responses

Create a `.turbo_stream.erb` template:

```ruby
# app/controllers/tasks_controller.rb
class TasksController < ApplicationController
  def create
    @task = @project.tasks.create!(task_params)
    respond_to do |format|
      format.turbo_stream
      format.html { redirect_to @project }
    end
  end
end
```

The template performs multiple DOM operations:

```erb
<!-- tasks/create.turbo_stream.erb -->
<%= turbo_stream.append "tasks", @task %>
<%= turbo_stream.update "task_count", html: pluralize(@project.tasks.count, "task") %>
<%= turbo_stream.replace "new_task_form", partial: "tasks/form", locals: { task: Task.new(project: @project) } %>
```

One response appends the task, updates the counter, and resets the form. The page never reloads.

### Inline Stream Responses

For simple cases, skip the template and render inline:

```ruby
def destroy
  @task = Task.find(params[:id])
  @task.destroy
  respond_to do |format|
    format.turbo_stream { render turbo_stream: turbo_stream.remove(dom_id(@task)) }
    format.html { redirect_to @project }
  end
end
```

The inline style is clean for single-action responses like removing an element.

### Combining with Frames

Stream responses and frames complement each other:

```erb
<!-- A form inside a frame that responds with streams -->
<%= turbo_frame_tag "new_comment" do %>
  <%= form_with model: [@post, Comment.new] do |f| %>
    <%= f.text_area :body %>
    <%= f.submit "Add Comment", data: { turbo_submits_with: "Posting..." } %>
  <% end %>
<% end %>
```

The controller responds with streams that update outside the frame:

```erb
<%= turbo_stream.append "comments", @comment %>
<%= turbo_stream.replace "new_comment", partial: "comments/form" %>
```

Streams can update any element on the page, even those outside the originating frame. This makes them more powerful than frames for multi-area updates.

## Common Pitfalls
1. **Forgetting HTML fallback** — Without `format.html`, the form won't work with JavaScript disabled or if Turbo fails to load.
2. **Target element missing** — Stream actions targeting non-existent IDs silently fail. Ensure target elements are in the DOM.

## Best Practices
1. **Use templates for multi-action responses** — When updating multiple parts of the page, a `.turbo_stream.erb` template is cleaner than inline rendering.
2. **Use inline for single actions** — For simple remove or replace operations, inline rendering keeps the code concise.

## Summary
- Controllers can respond with `format.turbo_stream` for targeted DOM updates.
- Stream templates (`.turbo_stream.erb`) support multiple actions per response.
- Inline rendering works well for single-action responses.
- Streams can update any element on the page, including outside the originating frame.
- Always include HTML fallback for progressive enhancement.

## Code Examples

**Inline stream response combining remove and update actions in a single response**

```ruby
# Inline stream response for deletion
def destroy
  @task = Task.find(params[:id])
  @task.destroy
  respond_to do |format|
    format.turbo_stream {
      render turbo_stream: [
        turbo_stream.remove(dom_id(@task)),
        turbo_stream.update("task_count",
          html: pluralize(@project.tasks.count, "task"))
      ]
    }
    format.html { redirect_to @project }
  end
end
```


## Resources

- [Turbo Streams Reference](https://turbo.hotwired.dev/reference/streams) — API reference for Turbo Stream actions and response format

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*