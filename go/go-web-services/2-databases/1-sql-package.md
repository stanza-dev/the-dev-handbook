---
source_course: "go-web-services"
source_lesson: "go-web-services-sql-package"
---

# database/sql Basics

## Introduction
Go provides a universal database interface through the `database/sql` package. It works with any SQL database via pluggable drivers, giving you connection pooling and prepared statements out of the box.

## Key Concepts
- **sql.Open()**: Initializes a connection pool and validates the DSN string, but does not establish any TCP connections.
- **sql.DB**: A connection pool (not a single connection) that manages multiple concurrent database connections automatically.
- **Driver import**: Drivers like `pgx` (Postgres) are imported for side effects only (`_ "github.com/jackc/pgx/v5/stdlib"`) to register themselves with `database/sql`.
- **db.Ping()**: Verifies actual connectivity by opening a real connection to the database.

## Real World Context
Every Go backend that talks to a relational database uses `database/sql`. Understanding that `sql.DB` is a pool — not a connection — is critical for writing concurrent services that handle thousands of requests without exhausting database connections.

## Deep Dive

Connect to a database by importing the driver and calling `sql.Open` with the driver name and DSN. The recommended PostgreSQL driver is `pgx/v5` (`github.com/jackc/pgx/v5`). Its `stdlib` sub-package registers a `database/sql` driver, giving you pgx performance with the standard interface. The older `lib/pq` driver is in maintenance mode and should only be used in legacy codebases.

```go
import (
    "database/sql"
    _ "github.com/jackc/pgx/v5/stdlib"
)

func main() {
    db, err := sql.Open("pgx", "postgres://user:pass@localhost/dbname?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    if err := db.Ping(); err != nil {
        log.Fatal(err)
    }
}
```

The `sql.Open` call only validates the DSN — no network connection happens yet. Call `db.Ping()` to verify actual connectivity.

Configure the connection pool to match your workload.

```go
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(25)
db.SetConnMaxLifetime(5 * time.Minute)
```

These settings control how many connections the pool maintains and how long they live. Tuning them prevents connection exhaustion under load.

## Common Pitfalls
1. **Assuming `sql.Open` connects immediately** — It only validates the DSN. Your app may start successfully but fail on the first query if the database is unreachable.
2. **Not configuring pool limits** — The default `MaxOpenConns` is unlimited, which can overwhelm your database under high traffic.

## Best Practices
1. **Always call `db.Ping()` after `sql.Open()`** — This verifies connectivity at startup instead of discovering issues at runtime.
2. **Set `MaxOpenConns` to match your database limits** — A typical starting point is 25 connections for a single service instance.

## Summary
- `sql.Open()` initializes the pool but does not connect — use `db.Ping()` to verify.
- `sql.DB` is a thread-safe connection pool, not a single connection.
- Configure `MaxOpenConns`, `MaxIdleConns`, and `ConnMaxLifetime` for production workloads.

## Code Examples

**A single-row query using QueryRow with parameterized placeholder — sql.ErrNoRows distinguishes 'not found' from actual errors**

```go
var name string
err := db.QueryRow("SELECT name FROM users WHERE id = $1", 1).Scan(&name)
if err == sql.ErrNoRows {
    // Handle not found
} else if err != nil {
    log.Fatal(err)
}
```


## Resources

- [database/sql Package](https://pkg.go.dev/database/sql) — Official Go documentation for the database/sql package
- [Accessing Databases](https://go.dev/doc/database/) — Official Go guide for working with relational databases

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*