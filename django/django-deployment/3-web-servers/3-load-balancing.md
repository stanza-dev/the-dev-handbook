---
source_course: "django-deployment"
source_lesson: "django-deployment-load-balancing"
---

# Load Balancing

## Introduction

Load balancing distributes traffic across multiple servers for reliability and scalability.

## Key Concepts

**Upstream**: Group of backend servers in Nginx.

**Load Balancing Methods**: Round-robin, least connections, IP hash.

**Health Checks**: Automatically remove unhealthy servers.

## Deep Dive

### Nginx Load Balancing

```nginx
upstream django {
    # Round-robin (default)
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003;
    
    # Or least connections
    least_conn;
    
    # Or IP hash (session persistence)
    ip_hash;
}

server {
    location / {
        proxy_pass http://django;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Weighted Servers

```nginx
upstream django {
    server 127.0.0.1:8001 weight=3;  # Gets 3x traffic
    server 127.0.0.1:8002;
    server 127.0.0.1:8003 backup;     # Only when others fail
}
```

## Best Practices

1. **Use least_conn for varying request times**: Better distribution.
2. **Add health checks**: Remove failed servers automatically.
3. **Consider session persistence**: Use ip_hash if needed.

## Summary

Nginx load balancing distributes traffic across multiple Gunicorn instances. Use least_conn for most applications and add backup servers for reliability.

## Resources

- [Nginx Load Balancing](https://nginx.org/en/docs/http/load_balancing.html) — Nginx load balancing guide

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*