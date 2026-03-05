---
source_course: "rails-foundations"
source_lesson: "rails-foundations-csrf-security"
---

# CSRF Protection and Security Basics

## Introduction

Rails includes several security features enabled by default. The most important is Cross-Site Request Forgery (CSRF) protection, which prevents attackers from tricking users into submitting malicious requests.

## Key Concepts

- **CSRF (Cross-Site Request Forgery)**: An attack where a malicious site tricks a user's browser into making unwanted requests to your app.
- **Authenticity token**: A unique token embedded in every form that Rails verifies on submission.
- **`protect_from_forgery`**: Enabled by default, it checks the authenticity token on non-GET requests.
- **Secure cookies**: Rails signs and encrypts session cookies by default, preventing tampering.

## Real World Context

Without CSRF protection, an attacker could create a page with a hidden form that transfers money from a logged-in user's account. Rails' CSRF token prevents this because the attacker cannot guess the token.

## Deep Dive

### How CSRF Protection Works

Rails embeds a unique token in every form:

```erb
<%= form_with model: @article do |f| %>
  <%# Rails automatically adds a hidden field: %>
  <%# <input type="hidden" name="authenticity_token" value="abc123..." /> %>
  <%= f.text_field :title %>
  <%= f.submit %>
<% end %>
```

When submitted, Rails checks that the token matches the one stored in the session.

### The Default Protection

```ruby
class ApplicationController < ActionController::Base
  # protect_from_forgery is enabled by default in Rails 8.1
end
```

It checks the authenticity token on every POST, PUT, PATCH, and DELETE request.

### Session Security

Rails encrypts session data by default:

```ruby
session[:user_id] = user.id        # Stored securely
session[:user_id]                   # Retrieved and verified

cookies.encrypted[:preference] = "dark_mode"
cookies.encrypted[:preference]  # => "dark_mode"
```

The encryption key comes from `Rails.application.credentials.secret_key_base`. Never expose this value.

## Common Pitfalls

- **Disabling CSRF for convenience**: Never use `skip_forgery_protection` for browser requests. It is only safe for API-only endpoints with token auth.
- **Exposing `secret_key_base`**: Keep it in credentials or environment variables, never in source code.
- **Using `html_safe` carelessly**: Marking user input as `html_safe` bypasses XSS protection.

## Best Practices

- Let Rails manage CSRF tokens automatically.
- Use `Rails.application.credentials` for all secrets.
- Sanitize user-generated HTML with `sanitize()` before rendering.

## Summary

- Rails includes CSRF protection by default via authenticity tokens in every form.
- Session cookies are encrypted and signed, preventing tampering.
- `protect_from_forgery` is enabled by default in `ActionController::Base`.
- Never disable CSRF protection for browser-facing controllers.
- Use `Rails.application.credentials` for secrets.

## Code Examples

**Rails security defaults — CSRF protection is automatic, sessions are encrypted, and cookies can be individually encrypted.**

```ruby
# Rails automatically protects against CSRF
class ApplicationController < ActionController::Base
  # protect_from_forgery is enabled by default
end

# Sessions are encrypted automatically
session[:user_id] = user.id

# Encrypted cookies for sensitive data
cookies.encrypted[:preference] = "dark_mode"
cookies.encrypted[:preference]  # => "dark_mode"
```


## Resources

- [Rails Security Guide](https://guides.rubyonrails.org/security.html) — Comprehensive guide to Rails security features including CSRF, XSS, and session security

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*