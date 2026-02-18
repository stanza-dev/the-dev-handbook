---
source_course: "go-web-services"
source_lesson: "go-web-services-health-checks"
---

# Health Endpoints

Kubernetes and load balancers need health endpoints.

## Liveness Probe

Is the application running?

```go
mux.HandleFunc("GET /healthz", func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("ok"))
})
```

## Readiness Probe

Is the application ready to receive traffic?

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

## Kubernetes Configuration

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

## Code Examples

**Detailed Health Check**

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


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*