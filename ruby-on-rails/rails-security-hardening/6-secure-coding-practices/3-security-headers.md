---
source_course: "rails-security-hardening"
source_lesson: "rails-security-security-headers"
---

# Security Headers

## Introduction
HTTP security headers provide defense-in-depth against various attacks. They instruct browsers to enable security features like HTTPS enforcement, click-jacking prevention, and content type validation.

## Key Concepts
- **X-Frame-Options**: Prevents your page from being embedded in iframes, blocking clickjacking attacks.
- **Strict-Transport-Security (HSTS)**: Forces browsers to use HTTPS for all future requests to your domain.
- **Permissions-Policy**: Controls which browser features (camera, microphone, geolocation) your page can access.
- **Referrer-Policy**: Controls how much referrer information is included in requests to other sites.

## Real World Context
Security headers are a low-effort, high-impact security measure. They take minutes to configure but protect against entire categories of attacks. Security scanning tools like Mozilla Observatory grade your site based on these headers.

## Deep Dive

### Default Rails Headers

Rails sets several security headers by default:

```ruby
# Default headers set by Rails:
# X-Frame-Options: SAMEORIGIN (prevents clickjacking)
# X-XSS-Protection: 0 (disabled — CSP is the modern replacement)
# X-Content-Type-Options: nosniff
# X-Permitted-Cross-Domain-Policies: none
```

These defaults provide baseline protection without any configuration.

### Custom Headers

Configure additional security headers in your application:

```ruby
# config/application.rb
config.action_dispatch.default_headers = {
  'X-Frame-Options' => 'DENY',
  'X-Content-Type-Options' => 'nosniff',
  'X-Permitted-Cross-Domain-Policies' => 'none',
  'Referrer-Policy' => 'strict-origin-when-cross-origin',
  'Permissions-Policy' => 'geolocation=(), microphone=(), camera=()'
}
```

Changing `X-Frame-Options` from `SAMEORIGIN` to `DENY` blocks all iframe embedding, including from your own domain.

### HSTS (Strict Transport Security)

Force HTTPS with HSTS:

```ruby
# config/environments/production.rb
config.force_ssl = true
config.ssl_options = {
  hsts: {
    expires: 1.year,
    subdomains: true,
    preload: true
  }
}
```

Once a browser receives an HSTS header, it will refuse to connect via HTTP for the specified duration. The `preload` option allows your domain to be included in browser preload lists.

### Per-Controller Headers

Set more restrictive headers for sensitive areas:

```ruby
class AdminController < ApplicationController
  before_action :set_security_headers

  private

  def set_security_headers
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['Cache-Control'] = 'no-store'
  end
end
```

The admin area uses `Cache-Control: no-store` to prevent sensitive data from being cached by browsers or proxies.

## Common Pitfalls
1. **Setting HSTS before your HTTPS setup is complete** — HSTS is hard to undo. If your SSL certificate expires, users cannot access your site at all. Start with a short expiry.
2. **Missing headers on API responses** — Security headers should be set on all responses, including JSON API responses, to prevent embedding attacks.

## Best Practices
1. **Use security header scanning tools** — Run Mozilla Observatory or SecurityHeaders.com to verify your configuration.
2. **Start HSTS with a short max-age** — Use `1.week` initially, then increase to `1.year` once you confirm HTTPS works correctly.

## Summary
- Rails sets baseline security headers by default.
- Configure additional headers like Referrer-Policy and Permissions-Policy.
- HSTS forces HTTPS and should be deployed carefully with incremental max-age.
- Set stricter headers for sensitive areas like admin panels.
- Scan your headers with tools like Mozilla Observatory.

## Code Examples

**Production security headers — HSTS forces HTTPS, X-Frame-Options blocks clickjacking, Permissions-Policy restricts browser APIs**

```ruby
# config/environments/production.rb
config.force_ssl = true
config.ssl_options = {
  hsts: { expires: 1.year, subdomains: true, preload: true }
}

# config/application.rb
config.action_dispatch.default_headers = {
  'X-Frame-Options' => 'DENY',
  'Referrer-Policy' => 'strict-origin-when-cross-origin',
  'Permissions-Policy' => 'geolocation=(), microphone=(), camera=()'
}
```


## Resources

- [Rails Default Security Headers](https://guides.rubyonrails.org/security.html#default-headers) — Official Rails guide on default security headers
- [Mozilla Observatory](https://observatory.mozilla.org/) — Free tool to scan and grade your site's security headers

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*