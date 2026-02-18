---
source_course: "go-architecture"
source_lesson: "go-architecture-config-management"
---

# Application Configuration

Hardcoding values is an anti-pattern. Go apps typically read config from:
1.  **Environment Variables:** Best for containerized apps.
2.  **Flags:** Good for CLI tools.
3.  **Files:** JSON/YAML/TOML.

## The `os` and `flag` packages

```go
import "flag"
import "os"

func main() {
    port := flag.Int("port", 8080, "server port")
    flag.Parse()

    dbHost := os.Getenv("DB_HOST")
    if dbHost == "" {
        dbHost = "localhost"  // Default
    }
}
```

## Configuration Struct Pattern

```go
type Config struct {
    Port     int    `env:"PORT" default:"8080"`
    DBHost   string `env:"DB_HOST" required:"true"`
    LogLevel string `env:"LOG_LEVEL" default:"info"`
}
```

Advanced apps use libraries like `viper` or `envconfig` to read from multiple sources with precedence.

## Code Examples

**Config Struct**

```go
type Config struct {
    Port     int
    Database string
    Debug    bool
}

func LoadConfig() *Config {
    return &Config{
        Port:     getEnvInt("PORT", 8080),
        Database: getEnv("DATABASE_URL", "postgres://localhost/app"),
        Debug:    getEnvBool("DEBUG", false),
    }
}
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*