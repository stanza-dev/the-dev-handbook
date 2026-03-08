---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-advanced-frame-patterns"
---

# Frame Navigation and Targeting

## Introduction
By default, links inside a Turbo Frame only update that frame. Turbo provides targeting controls to direct navigation to any frame or break out entirely for full-page navigation.

## Key Concepts
- **Frame Scoping**: Links inside a frame automatically target that frame.
- **`data-turbo-frame`**: Overrides the default frame target on links/forms.
- **`_top` Target**: Special value for full-page navigation, escaping frame scope.
- **Frame `target` attribute**: Default target for all links inside a frame.

## Real World Context
Frame targeting enables tabbed interfaces, sidebar navigation updating a main area, and search forms updating result lists — all without JavaScript.

## Deep Dive
### Default Scoping

```erb
<%= turbo_frame_tag "task_42" do %>
  <p>Buy groceries</p>
  <%= link_to "Edit", edit_task_path(42) %>  <!-- updates only task_42 -->
<% end %>
```

Clicking Edit replaces only this frame. Everything else stays untouched.

### Cross-Frame Targeting

Links can target any frame by ID:

```erb
<nav>
  <%= link_to "Overview", product_overview_path(@product),
      data: { turbo_frame: "tab_content" } %>
  <%= link_to "Reviews", product_reviews_path(@product),
      data: { turbo_frame: "tab_content" } %>
</nav>
<%= turbo_frame_tag "tab_content" do %>
  <%= render "overview" %>
<% end %>
```

Tab links outside the frame target it by ID, creating a tabbed interface with zero JavaScript.

### Default Target on Frame

Set a default target for all links in a frame:

```erb
<%= turbo_frame_tag "sidebar", target: "main_content" do %>
  <ul>
    <li><%= link_to "Dashboard", dashboard_path %></li>
    <li><%= link_to "Settings", settings_path %></li>
  </ul>
<% end %>
<%= turbo_frame_tag "main_content" do %>
  <%= yield %>
<% end %>
```

Every link in the sidebar updates `main_content` without per-link attributes.

### URL Updates

Frame navigations don't update the browser URL by default. Add `turbo_action` for bookmarkable URLs:

```erb
<%= link_to "Electronics", products_path(category: "electronics"),
    data: { turbo_frame: "product_list", turbo_action: "advance" } %>
```

## Common Pitfalls
1. **Unintended frame scoping** — Links inside frames only update that frame. Use `_top` when you want full-page navigation.
2. **Missing frame in response** — If the targeted frame doesn't exist in the response, Turbo shows a Content Missing error.

## Best Practices
1. **Use `target` on the frame element** — When all links should navigate the same target, set it on the frame.
2. **Add `turbo_action: "advance"` for bookmarkable navigations** — Tab selections, filters, and pagination should update the URL.

## Summary
- Links inside frames automatically scope to that frame.
- `data-turbo-frame="_top"` breaks out for full-page navigation.
- Links can target any frame by specifying its ID.
- The `target` attribute on a frame sets a default for all its links.
- `data-turbo-action="advance"` updates the browser URL.

## Code Examples

**A zero-JS tabbed interface built with frame targeting and URL-updating navigation**

```ruby
# A tabbed interface — zero JavaScript
# <nav>
#   <%= link_to "Details", product_details_path(@product),
#       data: { turbo_frame: "tab_panel", turbo_action: "advance" } %>
#   <%= link_to "Reviews", product_reviews_path(@product),
#       data: { turbo_frame: "tab_panel", turbo_action: "advance" } %>
# </nav>
# <%= turbo_frame_tag "tab_panel" do %>
#   <%= render "details" %>
# <% end %>
```


## Resources

- [Turbo Frames Handbook](https://turbo.hotwired.dev/handbook/frames) — Official handbook on frame navigation and targeting

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*