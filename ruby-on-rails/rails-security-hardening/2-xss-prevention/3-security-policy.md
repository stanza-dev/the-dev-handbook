---
source_course: "rails-security-hardening"
source_lesson: "rails-security-content-security-policy"
---

# Content Security Policy

Content Security Policy (CSP) is an HTTP header that tells browsers which sources of content to trust.

## Configuring CSP in Rails

Rails 7+ includes CSP support:

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

## CSP Directives

- `default-src`: Fallback for other directives
- `script-src`: Valid sources for JavaScript
- `style-src`: Valid sources for stylesheets
- `img-src`: Valid sources for images
- `connect-src`: Valid URLs for fetch, WebSocket, etc.
- `frame-src`: Valid sources for frames

## Nonces for Inline Scripts

CSP blocks inline scripts by default. Use nonces to allow specific ones:

```ruby
# In controller or layout
<%= tag.script nonce: content_security_policy_nonce do %>
  console.log('This inline script is allowed');
<% end %>
```

With CSP configuration:
```ruby
policy.script_src :self, :nonce
```

## Report-Only Mode

Test CSP without breaking your site:

```ruby
config.content_security_policy_report_only = true
```

## Per-Action CSP

```ruby
class PostsController < ApplicationController
  content_security_policy do |policy|
    policy.script_src :self, 'https://maps.googleapis.com'
  end, only: :show
end
```

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*