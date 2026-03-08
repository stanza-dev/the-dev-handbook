---
source_course: "rails-security-hardening"
source_lesson: "rails-security-content-security-policy"
---

# Content Security Policy

## Introduction
Content Security Policy (CSP) is an HTTP header that tells browsers which sources of content to trust. Even if an XSS vulnerability exists in your application, CSP can prevent the injected script from executing.

## Key Concepts
- **CSP Header**: An HTTP response header that defines a policy for allowed content sources.
- **Directive**: A rule within CSP that controls a specific resource type (scripts, styles, images, etc.).
- **Nonce**: A unique, random value generated per request that allows specific inline scripts.
- **Report-Only Mode**: A CSP mode that logs violations without blocking content, useful for testing.

## Real World Context
CSP is your last line of defense against XSS. Major sites like GitHub, Google, and Facebook all use strict CSP policies. Rails has built-in CSP support, making it straightforward to configure.

## Deep Dive

### Configuring CSP in Rails

Rails includes CSP support out of the box:

```ruby
# config/initializers/content_security_policy.rb
Rails.application.configure do
  config.content_security_policy do |policy|
    policy.default_src :self
    policy.font_src    :self, 'https://fonts.gstatic.com'
    policy.img_src     :self, :data, 'https://cdn.example.com'
    policy.script_src  :self
    policy.style_src   :self, 'https://fonts.googleapis.com'

    # Report violations to your server
    policy.report_uri '/csp-violation-report'
  end
end
```

Each directive specifies which sources are trusted for that resource type. The `:self` keyword means the same origin as the page.

### Nonces for Inline Scripts

CSP blocks inline scripts by default. Use nonces to allow specific ones:

```erb
<%= tag.script nonce: content_security_policy_nonce do %>
  console.log('This inline script is allowed');
<% end %>
```

The nonce is a random value that changes on every request. Only scripts with the matching nonce are executed.

Configure CSP to accept nonces:

```ruby
policy.script_src :self, :nonce
```

This tells the browser to execute inline scripts only if they carry the correct nonce value.

### Report-Only Mode

Test your CSP policy without breaking your site:

```ruby
config.content_security_policy_report_only = true
```

In report-only mode, violations are logged but not blocked. This lets you refine your policy before enforcing it.

### Per-Action CSP

Override CSP for specific controller actions:

```ruby
class PostsController < ApplicationController
  content_security_policy do |policy|
    policy.script_src :self, 'https://maps.googleapis.com'
  end, only: :show
end
```

This allows Google Maps scripts only on the show action, keeping a strict policy everywhere else.

## Common Pitfalls
1. **Using `unsafe-inline` for scripts** — This directive defeats the purpose of CSP entirely. Use nonces instead.
2. **Deploying CSP without report-only testing** — A misconfigured CSP can break your entire site. Always test in report-only mode first.

## Best Practices
1. **Start with a strict policy** — Begin with `default_src :self` and add exceptions only as needed.
2. **Monitor violation reports** — Set up a reporting endpoint to catch violations in production and detect potential attacks.

## Summary
- CSP is an HTTP header that restricts which content sources browsers trust.
- Rails has built-in CSP support via an initializer.
- Use nonces for inline scripts instead of `unsafe-inline`.
- Test policies in report-only mode before enforcing them.
- Per-action CSP overrides allow exceptions for specific pages.

## Code Examples

**A strict CSP configuration that only allows same-origin resources and nonce-authenticated inline scripts**

```ruby
# config/initializers/content_security_policy.rb
Rails.application.configure do
  config.content_security_policy do |policy|
    policy.default_src :self
    policy.script_src  :self, :nonce
    policy.style_src   :self, 'https://fonts.googleapis.com'
    policy.report_uri  '/csp-violation-report'
  end
end
```


## Resources

- [Rails Content Security Policy Guide](https://guides.rubyonrails.org/security.html#content-security-policy) — Official Rails guide on configuring Content Security Policy

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*