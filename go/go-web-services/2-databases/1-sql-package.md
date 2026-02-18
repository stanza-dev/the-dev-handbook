---
source_course: "go-web-services"
source_lesson: "go-web-services-sql-package"
---

# Connecting to Databases

Go uses `database/sql` as a generic interface, with specific drivers (like `lib/pq` for Postgres) imported for side effects.

```go
import (
    "database/sql"
    _ "github.com/lib/pq"
)

func main() {
    db, err := sql.Open("postgres", "postgres://user:pass@localhost/dbname?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    
    // Verify connection
    if err := db.Ping(); err != nil {
        log.Fatal(err)
    }
}
```

## Connection Pool
`sql.DB` is not a connection; it's a pool. Configure it:

```go
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(25)
db.SetConnMaxLifetime(5 * time.Minute)
```

## Code Examples

**Querying a Row**

```go
var name string
err := db.QueryRow("SELECT name FROM users WHERE id = $1", 1).Scan(&name)
if err == sql.ErrNoRows {
    // Handle not found
} else if err != nil {
    log.Fatal(err)
}
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*