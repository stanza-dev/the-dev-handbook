---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-load-balancing"
---

# Load Balancing

## Introduction

Load balancing distributes traffic across multiple application instances. Combined with stateless design and health checks, it enables high availability, fault tolerance, and horizontal scaling.

## Key Concepts

- **Load Balancer**: Distributes requests across servers
- **Health Checks**: Verify instance availability
- **Sticky Sessions**: Route user to same instance (avoid if possible)
- **Round Robin**: Distribute requests evenly

## Real World Context

Load balancing provides:
- High availability (instance failure tolerance)
- Horizontal scalability
- Geographic distribution
- SSL termination

## Deep Dive

### Docker Compose with Nginx

```yaml
# docker-compose.yml
version: '3.8'
services:
  app1:
    build: .
    environment:
      - NODE_ENV=production

  app2:
    build: .
    environment:
      - NODE_ENV=production

  nginx:
    image: nginx:alpine
    ports:
      - '80:80'
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app1
      - app2
```

### Nginx Configuration

```nginx
upstream nestjs {
    least_conn;  # Least connections algorithm
    server app1:3000 weight=1;
    server app2:3000 weight=1;
    keepalive 64;
}

server {
    listen 80;

    location / {
        proxy_pass http://nestjs;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }

    location /health {
        proxy_pass http://nestjs/health;
        proxy_connect_timeout 5s;
        proxy_read_timeout 5s;
    }
}
```

### Trust Proxy Settings

```typescript
import { NestFactory } from '@nestjs/core';
import { NestExpressApplication } from '@nestjs/platform-express';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);

  // Trust proxy headers (X-Forwarded-For, etc.)
  app.set('trust proxy', 1);

  await app.listen(3000);
}
```

### Kubernetes Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nestjs-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nestjs-app
  template:
    metadata:
      labels:
        app: nestjs-app
    spec:
      containers:
      - name: nestjs-app
        image: myregistry/nestjs-app:latest
        ports:
        - containerPort: 3000
        livenessProbe:
          httpGet:
            path: /health/live
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: '256Mi'
            cpu: '250m'
          limits:
            memory: '512Mi'
            cpu: '500m'
---
apiVersion: v1
kind: Service
metadata:
  name: nestjs-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: nestjs-app
```

### Horizontal Pod Autoscaler

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nestjs-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nestjs-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### Blue-Green Deployment

```bash
# Deploy new version to 'green'
kubectl apply -f deployment-green.yaml

# Wait for green to be ready
kubectl rollout status deployment/nestjs-green

# Switch traffic from blue to green
kubectl patch service nestjs-service -p '{"spec":{"selector":{"version":"green"}}}'

# Delete old 'blue' deployment
kubectl delete deployment nestjs-blue
```

## Common Pitfalls

1. **Session affinity needed**: Design stateless instead.
2. **No health checks**: Unhealthy instances receive traffic.
3. **Slow startup**: Instances not ready receive traffic.

## Best Practices

- Design for statelessness
- Implement comprehensive health checks
- Use readiness probes for slow-starting apps
- Configure auto-scaling based on metrics
- Use blue-green or rolling deployments

## Summary

Load balancing distributes traffic for availability and scalability. Use Nginx or cloud load balancers, implement health checks, and configure Kubernetes for auto-scaling. Design stateless to avoid sticky sessions.

## Resources

- [Health Checks](https://docs.nestjs.com/recipes/terminus) — Health checks for load balancers

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*