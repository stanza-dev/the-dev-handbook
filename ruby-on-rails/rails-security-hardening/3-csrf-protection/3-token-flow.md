---
source_course: "rails-security-hardening"
source_lesson: "rails-security-csrf-token-flow"
---

# CSRF Token Lifecycle

## Introduction
Understanding the full lifecycle of CSRF tokens — from generation to verification — helps you debug issues and correctly integrate CSRF protection with JavaScript frameworks and single-page applications.

## Key Concepts
- **Token Generation**: Rails generates a new masked token for each request, derived from the session's base token.
- **Token Masking**: A technique that creates a unique-looking token for each request while still validating against the same base token.
- **Token Rotation**: The process of invalidating old tokens when the session changes, such as after login.

## Real World Context
When integrating Rails with React, Vue, or other JavaScript frameworks, developers often struggle with CSRF token handling. Understanding the token lifecycle makes debugging 422 errors straightforward.

## Deep Dive

### Token Generation

Rails stores a base CSRF token in the session and generates a masked version for each request:

```ruby
# Rails internally generates tokens like this:
# 1. Base token stored in session[:_csrf_token]
# 2. Masked token = mask XOR base_token (unique per request)
# 3. Masked token is embedded in forms and meta tags
```

The masking ensures that each page load produces a different-looking token, preventing BREACH attacks that exploit HTTP compression.

### Token Verification Flow

When a state-changing request arrives, Rails verifies the token:

```ruby
# Rails verification pseudocode:
# 1. Extract token from params[:authenticity_token] or X-CSRF-Token header
# 2. Unmask the submitted token
# 3. Compare with the base token from the session
# 4. If they match, the request is legitimate
# 5. If not, raise InvalidAuthenticityToken (or clear session)
```

This comparison uses constant-time string comparison to prevent timing attacks.

### SPA Integration Pattern

For single-page applications, fetch the token from the meta tag on page load:

```javascript
// Read the CSRF token once and use it for all requests
const csrfToken = document.querySelector('meta[name="csrf-token"]')?.content;

// Configure a fetch wrapper
function apiFetch(url, options = {}) {
  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'X-CSRF-Token': csrfToken,
      'Content-Type': 'application/json',
    },
  });
}

// Usage:
apiFetch('/posts', {
  method: 'POST',
  body: JSON.stringify({ title: 'Hello' }),
});
```

This pattern reads the token once and includes it in all subsequent requests.

## Common Pitfalls
1. **Caching pages with CSRF tokens** — If a page with an embedded token is cached and served to multiple users, the token will not match their sessions. Use `no-store` cache headers for pages with forms.
2. **Token mismatch after session reset** — Calling `reset_session` invalidates the CSRF token. Make sure the client gets a fresh token after login or logout.

## Best Practices
1. **Include csrf_meta_tags in your layout** — This ensures JavaScript always has access to a valid token.
2. **Handle 422 errors gracefully** — When a token is invalid, redirect to a login page or refresh the token rather than showing an error.

## Summary
- Rails generates masked CSRF tokens derived from a session-stored base token.
- Tokens are verified using constant-time comparison on every state-changing request.
- For SPAs, read the token from the meta tag and include it in fetch headers.
- Token mismatches after session changes are a common debugging issue.

## Code Examples

**A fetch wrapper that includes the CSRF token in all requests and handles token expiration gracefully**

```javascript
// SPA pattern: read CSRF token from meta tag and include in all requests
const csrfToken = document.querySelector('meta[name="csrf-token"]')?.content;

async function apiFetch(url, options = {}) {
  const response = await fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'X-CSRF-Token': csrfToken,
      'Content-Type': 'application/json',
    },
  });
  if (response.status === 422) {
    // Token expired — reload page to get fresh token
    window.location.reload();
  }
  return response;
}
```


## Resources

- [Rails ActionController — Request Forgery Protection](https://api.rubyonrails.org/classes/ActionController/RequestForgeryProtection.html) — API reference for Rails CSRF protection internals

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*