---
source_course: "rails-foundations"
source_lesson: "rails-foundations-form-helpers"
---

# Form Helpers

## Introduction

Rails provides powerful form helpers that generate HTML forms integrated with your models. They handle CSRF protection, HTTP method spoofing, and pre-filling fields automatically.

## Key Concepts

- **`form_with`**: The modern form builder that generates forms tied to models or custom URLs.
- **CSRF protection**: Rails automatically includes an authenticity token in every form to prevent cross-site request forgery.
- **Model-backed forms**: When given a model, `form_with` automatically sets the action URL, HTTP method, and pre-fills field values.
- **Form helpers**: Methods like `text_field`, `text_area`, `select`, `check_box` that generate properly named HTML input elements.

## Real World Context

Forms are the primary way users interact with web applications. Rails form helpers handle the tedious parts (CSRF tokens, proper naming, pre-filling) so you can focus on the user experience. They also integrate with model validations to display error messages.

## Deep Dive

### Model-Backed Forms

```erb
<%= form_with model: @article do |form| %>
  <div>
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>

  <div>
    <%= form.label :body %>
    <%= form.text_area :body, rows: 10 %>
  </div>

  <%= form.submit %>
<% end %>
```

Rails automatically:
- Sets the action URL (POST for new, PATCH for existing records)
- Pre-fills fields with model values
- Names fields correctly (`article[title]`, `article[body]`)
- Adds CSRF protection token

### Common Form Helpers

```erb
<%= form_with model: @user do |f| %>
  <%= f.text_field :name %>
  <%= f.email_field :email %>
  <%= f.password_field :password %>
  <%= f.number_field :age %>
  <%= f.text_area :bio, rows: 5 %>
  <%= f.select :role, ["Admin", "Editor", "User"] %>
  <%= f.collection_select :category_id, Category.all, :id, :name %>
  <%= f.check_box :active %>
  <%= f.date_field :birthday %>
  <%= f.file_field :avatar %>
  <%= f.submit "Save User" %>
<% end %>
```

### Displaying Validation Errors

```erb
<%= form_with model: @article do |form| %>
  <% if @article.errors.any? %>
    <div class="errors">
      <h2><%= pluralize(@article.errors.count, "error") %> prevented saving:</h2>
      <ul>
        <% @article.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
  <!-- form fields... -->
<% end %>
```

### Non-Model Forms

```erb
<%= form_with url: search_path, method: :get do |form| %>
  <%= form.text_field :query, placeholder: "Search..." %>
  <%= form.submit "Search" %>
<% end %>
```

## Common Pitfalls

- **Forgetting `form_with` defaults to remote (Turbo)**: In Rails with Turbo, forms submit via fetch by default. Add `local: true` if you need a traditional form submission.
- **Wrong HTTP method for updates**: `form_with model: @article` for an existing record automatically uses PATCH. Don't manually set POST.
- **Not displaying validation errors**: Always show `@model.errors` in the form so users know what to fix.

## Best Practices

- Use model-backed forms (`form_with model:`) whenever possible for automatic URL and method handling.
- Always display validation errors clearly near the form or the specific field.
- Use `collection_select` for dropdowns that populate from database records.

## Summary

- `form_with model: @record` generates forms with automatic URL, method, and CSRF protection.
- New records submit POST to the create action; existing records submit PATCH to update.
- Form helpers (`text_field`, `select`, `check_box`, etc.) generate properly named HTML inputs.
- Validation errors are displayed by iterating over `@model.errors.full_messages`.
- Non-model forms use `form_with url:` for custom endpoints.

## Code Examples

**form_with generates HTML forms with automatic CSRF protection, correct HTTP methods, and pre-filled values when backed by a model.**

```ruby
# Model-backed form
# <%= form_with model: @article do |form| %>
#   <%= form.label :title %>
#   <%= form.text_field :title %>
#   <%= form.label :body %>
#   <%= form.text_area :body %>
#   <%= form.submit %>
# <% end %>
#
# Rails auto-detects: POST for new records, PATCH for existing
```


## Resources

- [Action View Form Helpers](https://guides.rubyonrails.org/form_helpers.html) — Complete guide to Rails form helpers

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*