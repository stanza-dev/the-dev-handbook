---
source_course: "django-deployment"
source_lesson: "django-deployment-ssl-certificates"
---

# SSL/TLS Configuration

## Introduction

SSL/TLS encryption is mandatory for production web applications. It protects data in transit, prevents man-in-the-middle attacks, and is required for HTTP/2. Let's Encrypt provides free, automated certificates that can be set up in minutes with Certbot.

## Key Concepts

**TLS Certificate**: Cryptographic credential proving server identity.

**Let's Encrypt**: Free, automated Certificate Authority.

**Certbot**: Tool for obtaining and renewing certificates.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

## Deep Dive

### Installing Certbot

```bash
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d example.com -d www.example.com

# Test renewal
sudo certbot renew --dry-run
```

### Nginx SSL Configuration

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000" always;
}
```

### Django Settings

```python
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Use Let's Encrypt**: Free, automatic, widely trusted.
2. **Enable HTTP/2**: Better performance with SSL.
3. **Set up auto-renewal**: Certbot does this automatically.

## Summary

Use Let's Encrypt with Certbot for free SSL certificates. Configure Nginx for TLS 1.2/1.3 and enable HTTP/2 for better performance.

## Code Examples

**Let's Encrypt SSL certificate setup with Certbot and Nginx**

```bash
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```


## Resources

- [Let's Encrypt](https://letsencrypt.org/docs/) — Let's Encrypt documentation

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*