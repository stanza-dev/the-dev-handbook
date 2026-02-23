---
source_course: "django-deployment"
source_lesson: "django-deployment-nginx-configuration"
---

# Nginx Configuration

## Introduction

Nginx is the most common reverse proxy for Django production deployments. It sits in front of Gunicorn/Uvicorn, handling TLS termination, static file serving, request buffering, load balancing, and rate limiting — tasks that application servers are not optimized for.

## Key Concepts

- **Reverse Proxy**: A server that forwards client requests to backend application servers and returns the responses.
- **Upstream**: An Nginx directive defining a group of backend servers for load balancing.
- **TLS Termination**: Handling HTTPS encryption/decryption at the Nginx layer so Gunicorn receives plain HTTP.
- **Rate Limiting**: Restricting the number of requests per client to protect against abuse and DDoS attacks.

## Real World Context

In a typical production stack, Nginx handles thousands of concurrent connections efficiently while Gunicorn focuses on running Python code. Nginx serves static files 10-100x faster than Django/Gunicorn because it's optimized for file I/O. Let's Encrypt integration via Certbot makes free SSL certificates automatic. Large-scale Django deployments use Nginx's load balancing to distribute traffic across multiple Gunicorn instances.

## Deep Dive

Nginx acts as a reverse proxy in front of Gunicorn, handling SSL termination, static file serving, request buffering, and load balancing. This separation of concerns improves both security and performance.


## Introduction

Nginx serves as a reverse proxy, handling SSL, static files, and load balancing.

## Basic Configuration

```nginx
# /etc/nginx/sites-available/myproject

upstream django {
    server unix:/run/gunicorn/myproject.sock;
    # Or TCP: server 127.0.0.1:8000;
}

server {
    listen 80;
    server_name example.com www.example.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    
    # SSL certificates
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Logging
    access_log /var/log/nginx/myproject_access.log;
    error_log /var/log/nginx/myproject_error.log;
    
    # Max upload size
    client_max_body_size 10M;
    
    # Static files
    location /static/ {
        alias /var/www/myproject/staticfiles/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Media files
    location /media/ {
        alias /var/www/myproject/media/;
        expires 1M;
        add_header Cache-Control "public";
    }
    
    # Django app
    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;
        proxy_pass http://django;
    }
}
```

## Enable Site

```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

## SSL with Let's Encrypt

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d example.com -d www.example.com

# Auto-renewal (usually set up automatically)
sudo certbot renew --dry-run
```

## Load Balancing

```nginx
upstream django {
    least_conn;  # Load balancing method
    
    server 127.0.0.1:8001 weight=3;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003 backup;
}
```

## WebSocket Support (for ASGI)

```nginx
location /ws/ {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_pass http://django;
}
```

## Rate Limiting

```nginx
# In http block
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

# In location block
location /api/ {
    limit_req zone=api burst=20 nodelay;
    proxy_pass http://django;
}
```

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Not setting `client_max_body_size`** — The default is 1MB, which means file uploads larger than 1MB will fail with a 413 error. Set it based on your application's needs.
2. **Forgetting proxy headers** — Without `X-Forwarded-For` and `X-Forwarded-Proto`, Django can't determine the real client IP or whether the request was HTTPS.
3. **Not testing config before reload** — Always run `nginx -t` before `systemctl reload nginx`. A syntax error in config will prevent Nginx from reloading.

## Best Practices

1. **Use Unix sockets for Gunicorn** — `proxy_pass http://unix:/run/gunicorn.sock` is faster and more secure than TCP.
2. **Enable HTTP/2** — Add `http2` to the `listen` directive for multiplexed connections and better performance.
3. **Set appropriate cache headers for static files** — Use `expires 1y` with hashed filenames for immutable caching.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**Nginx reverse proxy configuration with SSL and static file serving**

```bash
# /etc/nginx/sites-available/myproject

upstream django {
    server unix:/run/gunicorn/myproject.sock;
    # Or TCP: server 127.0.0.1:8000;
}

server {
    listen 80;
    server_name example.com www.example.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    
    # SSL certificates
# ...
```


## Resources

- [Nginx Docs](https://nginx.org/en/docs/) — Official Nginx documentation

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*