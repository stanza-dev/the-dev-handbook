---
source_course: "rails-security-hardening"
source_lesson: "rails-security-understanding-csrf"
---

# Understanding CSRF

Cross-Site Request Forgery (CSRF) tricks authenticated users into performing unwanted actions.

## How CSRF Works

Imagine you're logged into your bank. An attacker creates a malicious page:

```html
<!-- On evil-site.com -->
<form action="https://yourbank.com/transfer" method="POST" id="evil">
  <input type="hidden" name="to" value="attacker-account">
  <input type="hidden" name="amount" value="10000">
</form>
<script>document.getElementById('evil').submit();</script>
```

When you visit this page, your browser automatically includes your bank's cookies, and the transfer happens!

## Rails CSRF Protection

Rails automatically protects against CSRF with tokens:

```ruby
class ApplicationController < ActionController::Base
  # Enabled by default in Rails
  protect_from_forgery with: :exception
end
```

## How It Works

1. Rails generates a unique token for each session
2. Token is embedded in forms via `form_with`/`form_for`
3. Token is included in the meta tag for JavaScript requests
4. Rails verifies the token on POST/PUT/PATCH/DELETE requests

## The Authenticity Token

```erb
<!-- Rails automatically adds this to forms -->
<%= form_with model: @post do |f| %>
  <!-- Hidden field added automatically -->
  <!-- <input type="hidden" name="authenticity_token" value="xyz..."> -->
  ...
<% end %>

<!-- Meta tag for JavaScript -->
<%= csrf_meta_tags %>
<!-- Renders: <meta name="csrf-token" content="xyz..."> -->
```

## AJAX Requests

Rails UJS automatically includes the token, but for custom JavaScript:

```javascript
const token = document.querySelector('meta[name="csrf-token"]').content;

fetch('/posts', {
  method: 'POST',
  headers: {
    'X-CSRF-Token': token,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ title: 'New Post' })
});
```

See [CSRF Security Guide](https://guides.rubyonrails.org/security.html#cross-site-request-forgery-csrf).

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*