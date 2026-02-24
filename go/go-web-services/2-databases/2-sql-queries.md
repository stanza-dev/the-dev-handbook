---
source_course: "go-web-services"
source_lesson: "go-web-services-sql-queries"
---

# Queries & Prepared Statements

## Introduction
Go's `database/sql` package offers three query methods for different use cases: `QueryRow` for single rows, `Query` for result sets, and `Prepare` for reusable statements. Choosing the right one affects both correctness and performance.

## Key Concepts
- **db.QueryRow()**: Executes a query expected to return at most one row, returning a `*sql.Row` that must be scanned.
- **db.Query()**: Executes a query that returns multiple rows as a `*sql.Rows` iterator.
- **rows.Close()**: Releases the database connection back to the pool — must always be deferred after `Query()`.
- **db.Prepare()**: Compiles a query once on the server, allowing efficient reuse with different parameters.

## Real World Context
Forgetting to close `Rows` is one of the most common connection leak bugs in Go services. Understanding the lifecycle of query results prevents pool exhaustion that manifests as mysterious timeouts under load.

## Deep Dive

For a single row, use `QueryRow` which returns exactly one row. Call `.Scan()` to read column values into variables.

```go
var user User
err := db.QueryRow("SELECT id, name FROM users WHERE id = $1", id).
    Scan(&user.ID, &user.Name)
```

If no row matches, `Scan` returns `sql.ErrNoRows`, which you should handle explicitly.

For multiple rows, use `Query` and iterate with `rows.Next()`.

```go
rows, err := db.Query("SELECT id, name FROM users WHERE active = $1", true)
if err != nil {
    return err
}
defer rows.Close()

var users []User
for rows.Next() {
    var u User
    if err := rows.Scan(&u.ID, &u.Name); err != nil {
        return err
    }
    users = append(users, u)
}
return rows.Err()
```

Always check `rows.Err()` after the loop — it catches errors that occurred during iteration.

Prepared statements compile the query once and reuse it with different parameters.

```go
stmt, err := db.Prepare("SELECT id, name FROM users WHERE id = $1")
if err != nil {
    return err
}
defer stmt.Close()

for _, id := range ids {
    var user User
    stmt.QueryRow(id).Scan(&user.ID, &user.Name)
}
```

This avoids re-parsing the SQL on each execution, which matters for high-frequency queries.

## Common Pitfalls
1. **Forgetting `defer rows.Close()`** — Leaked rows hold a database connection, eventually exhausting the pool and causing timeouts.
2. **Ignoring `rows.Err()`** — Iteration can stop early due to a network error. Without checking `rows.Err()`, you silently return partial results.

## Best Practices
1. **Always `defer rows.Close()` immediately after `Query()`** — This ensures the connection returns to the pool even if an error occurs mid-iteration.
2. **Use `Prepare` for batch operations** — When executing the same query in a loop, a prepared statement avoids repeated parsing overhead.

## Summary
- Use `QueryRow` for single results, `Query` for multiple rows, and `Prepare` for repeated queries.
- Always `defer rows.Close()` and check `rows.Err()` after iteration.
- Prepared statements improve performance for repeated queries with different parameters.

## Code Examples

**A complete multi-row query pattern — defer rows.Close() prevents connection leaks and rows.Err() catches mid-iteration failures**

```go
func getUsers(db *sql.DB) ([]User, error) {
    rows, err := db.Query("SELECT id, name, email FROM users")
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var users []User
    for rows.Next() {
        var u User
        if err := rows.Scan(&u.ID, &u.Name, &u.Email); err != nil {
            return nil, err
        }
        users = append(users, u)
    }
    return users, rows.Err()
}
```


## Resources

- [database/sql Package](https://pkg.go.dev/database/sql) — Official Go documentation for Query, QueryRow, and Prepare methods
- [Accessing Databases](https://go.dev/doc/database/) — Official Go guide covering query patterns and best practices

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*