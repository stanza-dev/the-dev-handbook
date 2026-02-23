---
source_course: "django-deployment"
source_lesson: "django-deployment-nginx-security"
---

# Nginx Security Headers

## Introduction

Hardening Nginx involves configuring security headers, hiding server information, rate limiting requests, and restricting access patterns. These layers of defense protect your Django application from common web attacks.

## Key Headers

```nginx
server {
    # Prevent clickjacking
    add_header X-Frame-Options "SAMEORIGIN" always;
    
    # Prevent MIME sniffing
    add_header X-Content-Type-Options "nosniff" always;
    
    # XSS protection (legacy browsers)
    add_header X-XSS-Protection "1; mode=block" always;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Referrer policy
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    
    # Content Security Policy
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'" always;
}
```

## Hide Server Information

```nginx
http {
    server_tokens off;  # Hide Nginx version
}
```

## Key Concepts

The key terms and concepts for this topic are introduced in the Deep Dive section below.


## Deep Dive

See the detailed technical content and code examples throughout this lesson.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Always use HSTS**: Force HTTPS.
2. **Set X-Frame-Options**: Prevent clickjacking.
3. **Hide server version**: Don't expose software versions.

## Summary

Add security headers in Nginx to protect against clickjacking, XSS, and other attacks. Enable HSTS and hide server version information.

## Code Examples

**Security headers configuration in Nginx for production hardening**

```bash
server {
    # Prevent clickjacking
    add_header X-Frame-Options "SAMEORIGIN" always;
    
    # Prevent MIME sniffing
    add_header X-Content-Type-Options "nosniff" always;
    
    # XSS protection (legacy browsers)
    add_header X-XSS-Protection "1; mode=block" always;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Referrer policy
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    
    # Content Security Policy
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'" always;
}
```


## Resources

- [Security Headers](https://securityheaders.com/) — Security headers checker

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*