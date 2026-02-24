---
source_course: "go-web-services"
source_lesson: "go-web-services-health-checks"
---

# Health Checks & Readiness

## Introduction
Health endpoints let orchestrators like Kubernetes know whether your service is alive and ready to receive traffic. They are the foundation of self-healing infrastructure and zero-downtime deployments.

## Key Concepts
- **Liveness probe**: Answers "Is the process alive?" — a failure triggers a container restart.
- **Readiness probe**: Answers "Can this instance handle traffic?" — a failure removes the instance from the load balancer.
- **Startup probe**: Answers "Has the application finished starting?" — prevents liveness checks from killing slow-starting apps.
- **Dependency checks**: Readiness probes typically verify database and cache connectivity before reporting ready.

## Real World Context
Without health checks, a crashed service keeps receiving traffic (no liveness probe) or an instance with a dead database connection gets requests it cannot serve (no readiness probe). Health endpoints are required for any containerized production deployment.

## Deep Dive

A liveness probe is a minimal endpoint that returns 200 if the process is running.

```go
mux.HandleFunc("GET /healthz", func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("ok"))
})
```

This endpoint should have no external dependencies — its only job is to confirm the process has not deadlocked.

A readiness probe checks external dependencies like the database.

```go
func readinessHandler(db *sql.DB) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        ctx, cancel := context.WithTimeout(r.Context(), 2*time.Second)
        defer cancel()

        if err := db.PingContext(ctx); err != nil {
            http.Error(w, "Database unavailable", http.StatusServiceUnavailable)
            return
        }
        w.WriteHeader(http.StatusOK)
        w.Write([]byte("ready"))
    }
}
```

The 2-second timeout prevents the health check itself from hanging if the database is unresponsive.

Kubernetes configures these probes in the pod spec.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

Kubernetes calls these endpoints periodically and reacts to failures automatically.

## Common Pitfalls
1. **Adding database checks to the liveness probe** — A database outage restarts all your pods, making the situation worse. Keep liveness probes dependency-free.
2. **No timeout on readiness checks** — A hanging database connection blocks the health check goroutine indefinitely.

## Best Practices
1. **Keep liveness probes trivial** — Return 200 unconditionally. Only check process health, not dependencies.
2. **Use readiness probes for dependency checks** — This removes unhealthy instances from the load balancer without restarting them.

## Summary
- Liveness probes check if the process is alive; readiness probes check if it can serve traffic.
- Keep liveness probes dependency-free to avoid cascading restarts.
- Use timeouts on readiness probes to prevent hanging health checks.

## Code Examples

**A detailed health check handler that reports per-dependency status — returns 'degraded' if any dependency fails while the process itself is alive**

```go
type HealthResponse struct {
    Status  string            `json:"status"`
    Checks  map[string]string `json:"checks"`
}

func healthHandler(db *sql.DB, cache *redis.Client) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        health := HealthResponse{
            Status: "ok",
            Checks: make(map[string]string),
        }

        if err := db.Ping(); err != nil {
            health.Status = "degraded"
            health.Checks["database"] = err.Error()
        } else {
            health.Checks["database"] = "ok"
        }

        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(health)
    }
}
```


## Resources

- [net/http Package](https://pkg.go.dev/net/http) — Official Go documentation for building HTTP endpoints used as health checks

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*