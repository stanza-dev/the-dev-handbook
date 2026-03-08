---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-hotwire-active-storage"
---

# Combining Frames and Streams

## Introduction
Turbo Frames and Turbo Streams solve different problems and are most powerful when combined. Frames handle in-place editing and scoped navigation. Streams handle multi-area updates and real-time broadcasts. Together, they can build complex interactions that rival SPAs.

## Key Concepts
- **Frame for navigation, Streams for side effects**: Use frames for the primary interaction (edit form) and streams for secondary updates (counters, lists, notifications).
- **Stream responses from frame submissions**: A form inside a frame can respond with streams that update areas outside the frame.
- **Broadcast + Frame**: Real-time broadcasts can update frame content or trigger frame reloads.

## Real World Context
Consider a project management app. Clicking "Edit" on a task opens an inline form (Turbo Frame). Saving the task updates the task display (frame), increments the completion counter (stream), and notifies other team members (broadcast). This multi-area, multi-user update pattern requires combining all three Turbo features.

## Deep Dive
### Frame Form with Stream Side Effects

A form inside a frame can trigger stream updates outside the frame:

```erb
<!-- tasks/show.html.erb -->
<h1><%= @project.name %></h1>
<p id="task_count"><%= pluralize(@project.tasks.completed.count, "task") %> completed</p>

<div id="tasks">
  <% @project.tasks.each do |task| %>
    <%= turbo_frame_tag dom_id(task) do %>
      <%= render task %>
    <% end %>
  <% end %>
</div>
```

The controller responds with streams instead of a redirect:

```ruby
def update
  @task = Task.find(params[:id])
  if @task.update(task_params)
    respond_to do |format|
      format.turbo_stream
      format.html { redirect_to @project }
    end
  else
    render :edit, status: :unprocessable_entity
  end
end
```

The stream template updates multiple areas:

```erb
<!-- tasks/update.turbo_stream.erb -->
<%= turbo_stream.replace dom_id(@task), @task %>
<%= turbo_stream.update "task_count",
    html: pluralize(@project.tasks.completed.count, "task") + " completed" %>
```

One response replaces the task in its frame AND updates the counter outside any frame.

### Real-Time Multi-User Updates

Combine model broadcasts with frame-based UI:

```ruby
class Task < ApplicationRecord
  belongs_to :project
  broadcasts_to :project
end
```

```erb
<!-- projects/show.html.erb -->
<%= turbo_stream_from @project %>

<div id="tasks">
  <% @project.tasks.each do |task| %>
    <%= turbo_frame_tag dom_id(task) do %>
      <%= render task %>
    <% end %>
  <% end %>
</div>
```

User A edits a task inline via the frame. The save broadcasts an update to all connected users. User B sees the task update in real-time without any page interaction.

## Common Pitfalls
1. **Stream actions targeting frame content** — If you broadcast a `replace` for an element inside a frame, it works, but the frame scoping rules don't apply to streams. Streams always target by ID regardless of frame context.
2. **Ordering issues** — Streams execute in order. If you remove and then append, ensure the target container exists before appending.

## Best Practices
1. **Use frames for user-initiated interactions** — Edit, navigate, filter. Frames give the user immediate feedback.
2. **Use streams for side effects and broadcasts** — Counter updates, list modifications, notifications. Streams handle the ripple effects.

## Summary
- Frames handle primary interactions (edit, navigate), streams handle side effects (counters, notifications).
- Form submissions inside frames can respond with streams that update outside the frame.
- Model broadcasts push real-time updates to all users via WebSockets.
- Streams always target by ID, ignoring frame scoping rules.

## Code Examples

**A stream template that updates the task, refreshes a counter, and prepends to an activity log — all from one form submission**

```ruby
# tasks/update.turbo_stream.erb
# Combines frame replacement with stream side effects:
<%= turbo_stream.replace dom_id(@task), @task %>
<%= turbo_stream.update "task_count",
    html: pluralize(@project.tasks.completed.count, "task") + " completed" %>
<%= turbo_stream.prepend "activity_log",
    partial: "activities/activity",
    locals: { activity: @task.activities.last } %>
```


## Resources

- [Turbo Handbook](https://turbo.hotwired.dev/handbook/introduction) — Complete Turbo handbook covering Drive, Frames, and Streams together

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*