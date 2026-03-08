---
source_course: "rails-security-hardening"
source_lesson: "rails-security-understanding-csrf"
---

# Understanding CSRF

## Introduction
Cross-Site Request Forgery (CSRF) tricks authenticated users into performing unwanted actions on a site where they are logged in. Unlike XSS which injects scripts, CSRF exploits the browser's automatic inclusion of cookies in requests.

## Key Concepts
- **CSRF Attack**: A forged request that uses a victim's authenticated session to perform unauthorized actions.
- **Authenticity Token**: A unique, unpredictable token that Rails embeds in forms and verifies on state-changing requests.
- **Same-Origin Policy**: A browser security mechanism that restricts how documents from one origin can interact with resources from another.

## Real World Context
CSRF attacks can transfer money, change email addresses, or modify account settings — all without the user's knowledge. Any state-changing action (POST, PUT, PATCH, DELETE) that relies solely on cookies for authentication is vulnerable.

## Deep Dive

### How CSRF Works

Imagine you are logged into your bank. An attacker creates a malicious page on a different site:

```html
<!-- On evil-site.com -->
<form action="https://yourbank.com/transfer" method="POST" id="evil">
  <input type="hidden" name="to" value="attacker-account">
  <input type="hidden" name="amount" value="10000">
</form>
<script>document.getElementById('evil').submit();</script>
```

When you visit this page, your browser automatically includes your bank's cookies with the forged request, and the transfer executes without your consent.

### Rails CSRF Protection

Rails automatically protects against CSRF with tokens:

```ruby
class ApplicationController < ActionController::Base
  # Enabled by default in Rails
  protect_from_forgery with: :exception
end
```

This tells Rails to verify an authenticity token on every non-GET request. If the token is missing or invalid, Rails raises `ActionController::InvalidAuthenticityToken`.

### How the Token Works

Rails generates a unique token for each session and embeds it in two places:

```erb
<!-- Rails automatically adds a hidden field to forms -->
<%= form_with model: @post do |f| %>
  <!-- Hidden field added automatically: -->
  <!-- <input type="hidden" name="authenticity_token" value="xyz..."> -->
  <%= f.text_field :title %>
  <%= f.submit %>
<% end %>

<!-- Meta tag for JavaScript requests -->
<%= csrf_meta_tags %>
<!-- Renders: <meta name="csrf-token" content="xyz..."> -->
```

The token in the form submission must match the token in the session. Since the attacker's page cannot read the token (blocked by same-origin policy), they cannot forge a valid request.

### AJAX Requests

For JavaScript fetch requests, include the token from the meta tag:

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

Rails checks the `X-CSRF-Token` header as an alternative to the form field, making it work seamlessly with JavaScript.

## Common Pitfalls
1. **Skipping CSRF for convenience** — Using `skip_before_action :verify_authenticity_token` broadly disables protection. Only skip it for specific endpoints with alternative authentication.
2. **Not including csrf_meta_tags** — Without the meta tag, JavaScript requests cannot include the token and will be rejected.

## Best Practices
1. **Keep protect_from_forgery enabled globally** — Only exempt specific endpoints that use alternative authentication (APIs with token auth, webhooks with signature verification).
2. **Use form_with for all forms** — It automatically includes the authenticity token. Manual `<form>` tags require manually adding the token.

## Summary
- CSRF exploits the browser's automatic cookie inclusion to forge authenticated requests.
- Rails protects against CSRF with per-session authenticity tokens.
- Tokens are embedded in forms automatically and available via meta tags for JavaScript.
- Never skip CSRF protection without providing an alternative authentication mechanism.

## Code Examples

**Rails CSRF protection setup — tokens are verified automatically on all state-changing requests**

```ruby
class ApplicationController < ActionController::Base
  # Enabled by default — raises exception on invalid token
  protect_from_forgery with: :exception
end

# Forms automatically include the token:
# <%= form_with model: @post do |f| %>
#   (hidden authenticity_token field added automatically)
# <% end %>

# JavaScript reads token from meta tag:
# const token = document.querySelector('meta[name="csrf-token"]').content;
```


## Resources

- [Rails Security Guide — CSRF](https://guides.rubyonrails.org/security.html#cross-site-request-forgery-csrf) — Official Rails guide on CSRF protection mechanisms

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*