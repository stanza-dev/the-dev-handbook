---
source_course: "go-web-services"
source_lesson: "go-web-services-graceful-shutdown"
---

# Graceful Shutdown

## Introduction
When a Go service is restarted or deployed, it receives a termination signal. Graceful shutdown ensures in-flight requests complete before the process exits, preventing broken responses and data corruption.

## Key Concepts
- **SIGTERM**: A signal sent by orchestrators (Kubernetes, Docker, systemd) requesting the process to shut down gracefully.
- **SIGINT**: The signal sent when pressing Ctrl+C — typically handled the same way as SIGTERM.
- **server.Shutdown(ctx)**: Stops accepting new connections and waits for active requests to complete, with a context deadline as the hard limit.
- **signal.Notify**: Registers a channel to receive specific OS signals, allowing the program to react.

## Real World Context
In Kubernetes, a pod receives SIGTERM before being killed. If your server does not handle this signal, active requests get abruptly terminated, causing 502 errors for users and potential data loss for in-progress writes.

## Deep Dive

The standard graceful shutdown pattern starts the server in a goroutine and blocks the main goroutine on a signal channel.

```go
func main() {
    srv := &http.Server{Addr: ":8080", Handler: router}

    go func() {
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatal(err)
        }
    }()

    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := srv.Shutdown(ctx); err != nil {
        log.Fatal("Server forced to shutdown:", err)
    }
    log.Println("Server exited gracefully")
}
```

`srv.Shutdown(ctx)` stops accepting new connections immediately but waits for active requests to finish. If they take longer than the context timeout (30 seconds here), the shutdown is forced.

## Common Pitfalls
1. **Not handling SIGTERM at all** — The process gets killed abruptly, dropping in-flight requests and potentially corrupting writes.
2. **Setting the timeout too short** — A 1-second timeout may kill long-running requests. Match the timeout to your longest expected request duration.

## Best Practices
1. **Always handle both SIGINT and SIGTERM** — SIGINT covers local development (Ctrl+C) and SIGTERM covers production deployments.
2. **Close other resources after server shutdown** — After `srv.Shutdown()`, close database connections, flush logs, and release other resources in order.

## Summary
- Catch SIGTERM and SIGINT with `signal.Notify` to trigger graceful shutdown.
- `server.Shutdown(ctx)` finishes active requests before stopping, with a timeout as the hard limit.
- Always clean up database connections and other resources after the server stops.

## Code Examples

**Signal handling for graceful shutdown — the context timeout ensures the process exits even if some requests are stuck**

```go
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
<-quit

ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()

if err := server.Shutdown(ctx); err != nil {
    log.Fatal("Forced shutdown:", err)
}
```


## Resources

- [net/http Package — Server.Shutdown](https://pkg.go.dev/net/http#Server.Shutdown) — Official Go documentation for the Shutdown method and its behavior

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*