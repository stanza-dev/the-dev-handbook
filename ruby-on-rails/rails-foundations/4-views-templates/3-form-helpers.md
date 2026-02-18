---
source_course: "rails-foundations"
source_lesson: "rails-foundations-form-helpers"
---

# Form Helpers

Rails provides powerful helpers to build forms that work seamlessly with your models.

## form_with

The modern way to create forms in Rails:

### Model-backed Forms

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

  <div>
    <%= form.label :published %>
    <%= form.check_box :published %>
  </div>

  <%= form.submit %>
<% end %>
```

Rails automatically:
- Sets the form action URL (POST for new, PATCH for existing)
- Pre-fills fields with model values
- Names fields correctly (article[title], article[body])
- Adds CSRF protection token

### Generated HTML

```html
<form action="/articles" method="post">
  <input type="hidden" name="authenticity_token" value="...">

  <div>
    <label for="article_title">Title</label>
    <input type="text" name="article[title]" id="article_title">
  </div>
  ...
</form>
```

## Common Form Helpers

```erb
<%= form_with model: @user do |f| %>
  <!-- Text inputs -->
  <%= f.text_field :name %>
  <%= f.email_field :email %>
  <%= f.password_field :password %>
  <%= f.telephone_field :phone %>
  <%= f.url_field :website %>
  <%= f.number_field :age %>

  <!-- Text areas -->
  <%= f.text_area :bio, rows: 5 %>

  <!-- Select dropdowns -->
  <%= f.select :role, ["Admin", "Editor", "User"] %>
  <%= f.select :country, options_for_select([["USA", "us"], ["Canada", "ca"]]) %>
  <%= f.collection_select :category_id, Category.all, :id, :name %>

  <!-- Checkboxes and radios -->
  <%= f.check_box :active %>
  <%= f.collection_check_boxes :tag_ids, Tag.all, :id, :name %>
  <%= f.collection_radio_buttons :status, [["Draft", "draft"], ["Published", "published"]], :last, :first %>

  <!-- Date and time -->
  <%= f.date_field :birthday %>
  <%= f.datetime_local_field :published_at %>

  <!-- Hidden and file -->
  <%= f.hidden_field :user_id %>
  <%= f.file_field :avatar %>

  <%= f.submit "Save User" %>
<% end %>
```

## Displaying Validation Errors

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

  <div class="<%= 'field-error' if @article.errors[:title].any? %>">
    <%= form.label :title %>
    <%= form.text_field :title %>
    <% @article.errors[:title].each do |error| %>
      <span class="error"><%= error %></span>
    <% end %>
  </div>

  <%= form.submit %>
<% end %>
```

## Non-Model Forms

```erb
<%= form_with url: search_path, method: :get do |form| %>
  <%= form.text_field :query, placeholder: "Search..." %>
  <%= form.submit "Search" %>
<% end %>
```

## Resources

- [Action View Form Helpers](https://guides.rubyonrails.org/form_helpers.html) — Complete guide to Rails form helpers

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*