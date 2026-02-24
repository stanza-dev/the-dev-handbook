---
source_course: "go-web-services"
source_lesson: "go-web-services-slog-package"
---

# Structured Logging (slog)

## Introduction
Go 1.21 introduced `log/slog`, a structured logging package in the standard library. Unlike `fmt.Printf` or the old `log` package, slog outputs machine-parseable key-value pairs that integrate with log aggregation systems.

## Key Concepts
- **Structured logging**: Logging with explicit key-value pairs instead of free-form strings, making logs searchable and filterable.
- **Log levels**: `Debug`, `Info`, `Warn`, `Error` — each represents increasing severity.
- **Handler**: The backend that formats log output. `slog.NewJSONHandler` produces JSON; `slog.NewTextHandler` produces human-readable text.
- **slog.With()**: Creates a child logger with pre-attached attributes, avoiding repetition across related log calls.

## Real World Context
In production, plain-text logs are nearly useless. Log aggregation tools (Datadog, Grafana Loki, CloudWatch) need structured fields to filter, search, and alert. Using slog from the start means your logs are production-ready without a third-party library.

## Deep Dive

The simplest usage passes key-value pairs after the message string.

```go
import "log/slog"

slog.Info("user login", "user_id", 42, "ip", "127.0.0.1")
```

This outputs structured data with `user_id` and `ip` as searchable fields.

Four log levels are available, each for a different severity.

*   `slog.Debug()` — detailed development information
*   `slog.Info()` — normal operational events
*   `slog.Warn()` — unexpected but recoverable situations
*   `slog.Error()` — failures that need attention

For production, configure a JSON handler to output machine-parseable logs.

```go
logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.LevelInfo,
}))
slog.SetDefault(logger)
```

This produces output like the following, which log aggregation tools can parse automatically.

```json
{"time":"2024-01-01T00:00:00Z","level":"INFO","msg":"user login","user_id":42,"ip":"127.0.0.1"}
```

The JSON format makes every field independently queryable in your logging platform.

## Common Pitfalls
1. **Using `fmt.Printf` for logging in production** — Free-form strings cannot be filtered or aggregated. Switch to slog for any code that will run in production.
2. **Logging at the wrong level** — Using `Error` for non-critical events creates alert fatigue. Reserve `Error` for actual failures.

## Best Practices
1. **Set the default logger at application startup** — Call `slog.SetDefault()` early so all packages use the same structured handler.
2. **Use JSON handler in production, text handler in development** — JSON is for machines; text is for humans reading terminal output.

## Summary
- `log/slog` (Go 1.21+) provides structured logging with key-value pairs.
- Use `slog.NewJSONHandler` for production and `slog.NewTextHandler` for development.
- Four log levels (Debug, Info, Warn, Error) help filter by severity.

## Code Examples

**Example JSON log output from slog.NewJSONHandler — every field is independently queryable in log aggregation tools**

```json
{"time":"2024-01-01T00:00:00Z","level":"INFO","msg":"user login","user_id":42,"ip":"127.0.0.1"}
```


## Resources

- [log/slog Package](https://pkg.go.dev/log/slog) — Official Go documentation for the structured logging package

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*