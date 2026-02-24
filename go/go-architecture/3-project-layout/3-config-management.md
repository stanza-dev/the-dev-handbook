---
source_course: "go-architecture"
source_lesson: "go-architecture-config-management"
---

# Configuration Management

## Introduction
Hardcoding values like ports, database URLs, and feature flags is an anti-pattern that makes deployment painful. Go applications typically read configuration from environment variables, command-line flags, or config files. Understanding these approaches lets you build twelve-factor apps that work across environments.

## Key Concepts
- **Environment Variables**: Key-value pairs set by the operating system or container runtime. Best for secrets and deployment-specific values.
- **Command-Line Flags**: Arguments parsed at startup using the `flag` package. Best for CLI tools and developer overrides.
- **Config Struct**: A struct that aggregates all configuration in one place with typed fields and defaults.

## Real World Context
Your application runs in development on your laptop, in staging on Kubernetes, and in production on a different cluster. Each environment has different database URLs, ports, and log levels. Environment variables let you configure all three without changing code.

## Deep Dive
The `flag` and `os` packages cover most needs:

```go
import (
    "flag"
    "os"
)

func main() {
    port := flag.Int("port", 8080, "server port")
    flag.Parse()

    dbHost := os.Getenv("DB_HOST")
    if dbHost == "" {
        dbHost = "localhost"  // Default
    }
}
```

A config struct pattern centralizes all settings:

```go
type Config struct {
    Port     int    `env:"PORT" default:"8080"`
    DBHost   string `env:"DB_HOST" required:"true"`
    LogLevel string `env:"LOG_LEVEL" default:"info"`
}
```

For complex applications, libraries like `viper` or `envconfig` can read from multiple sources (env vars, files, flags) with precedence rules. But for most services, the standard library is sufficient.

A helper function pattern keeps config loading clean:

```go
func LoadConfig() *Config {
    return &Config{
        Port:     getEnvInt("PORT", 8080),
        Database: getEnv("DATABASE_URL", "postgres://localhost/app"),
        Debug:    getEnvBool("DEBUG", false),
    }
}
```

## Common Pitfalls
1. **Scattering `os.Getenv` calls throughout the codebase** — Centralize config reading in one place so you can see all settings at a glance.
2. **Not providing defaults** — Always have sensible defaults for development so the app runs with zero configuration.

## Best Practices
1. **Use a single Config struct** — Load all config at startup, validate it, and pass it through dependency injection.
2. **Fail fast on missing required config** — If a required env var is missing, exit immediately with a clear error message.

## Summary
- Use environment variables for deployment-specific configuration.
- Use command-line flags for CLI tools and developer overrides.
- Centralize all config in a struct with defaults.
- Validate required configuration at startup and fail fast.

## Code Examples

**A centralized Config struct with helper functions that read environment variables and fall back to sensible defaults**

```go
type Config struct {
    Port     int
    Database string
    Debug    bool
}

// LoadConfig reads configuration from environment variables
// with sensible defaults for local development.
func LoadConfig() *Config {
    return &Config{
        Port:     getEnvInt("PORT", 8080),
        Database: getEnv("DATABASE_URL", "postgres://localhost/app"),
        Debug:    getEnvBool("DEBUG", false),
    }
}
```


## Resources

- [flag package documentation](https://pkg.go.dev/flag) — Standard library package for parsing command-line flags
- [The Twelve-Factor App — Config](https://12factor.net/config) — Industry-standard methodology for application configuration via environment variables

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*