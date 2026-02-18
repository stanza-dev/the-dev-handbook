---
source_course: "go-web-services"
source_lesson: "go-web-services-slog-package"
---

# log/slog (Go 1.21+)

Structured logging allows machines to parse your logs easily.

```go
import "log/slog"

slog.Info("user login", "user_id", 42, "ip", "127.0.0.1")
```

## Log Levels

*   `slog.Debug()`
*   `slog.Info()`
*   `slog.Warn()`
*   `slog.Error()`

## JSON Handler
For production, you typically want JSON output.

```go
logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.LevelInfo,
}))
slog.SetDefault(logger)
```

Output:
```json
{"time":"2024-01-01T00:00:00Z","level":"INFO","msg":"user login","user_id":42,"ip":"127.0.0.1"}
```

## Code Examples

**JSON Log Output**

```json
{"time":"2024-01-01T00:00:00Z","level":"INFO","msg":"user login","user_id":42,"ip":"127.0.0.1"}
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*