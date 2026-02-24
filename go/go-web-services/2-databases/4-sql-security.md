---
source_course: "go-web-services"
source_lesson: "go-web-services-sql-security"
---

# SQL Security & Best Practices

## Introduction
SQL injection remains one of the most dangerous web vulnerabilities. Go's `database/sql` package makes prevention easy with parameterized queries, but you must use them consistently and understand the related security toolbox.

## Key Concepts
- **Parameterized queries**: Placeholders like `$1` (Postgres) or `?` (MySQL) that safely bind user input, preventing SQL injection.
- **sql.NullString**: A type that correctly handles SQL NULL values, which Go's zero-value strings cannot represent.
- **context.WithTimeout**: Creates a context that cancels long-running queries after a deadline, preventing resource exhaustion.
- **Placeholder syntax**: Varies by database — PostgreSQL uses `$1, $2`, MySQL uses `?`, SQLite uses both.

## Real World Context
A single SQL injection vulnerability can expose your entire database. Using parameterized queries is non-negotiable in production. Additionally, query timeouts prevent a slow query from holding a connection forever, which can cascade into service-wide outages.

## Deep Dive

**Always use parameterized queries.** Never concatenate user input into SQL strings.

```go
// VULNERABLE - Never do this!
query := "SELECT * FROM users WHERE name = '" + input + "'"

// SAFE - Use placeholders
db.Query("SELECT * FROM users WHERE name = $1", input)
```

The parameterized version sends the query structure and values separately, making injection impossible.

Different databases use different placeholder syntax.

*   PostgreSQL: `$1`, `$2`, `$3`
*   MySQL: `?`, `?`, `?`
*   SQLite: `?` or `$1`

Handle nullable database columns with `sql.NullString` and related types.

```go
type User struct {
    ID    int
    Name  string
    Email sql.NullString
}

if user.Email.Valid {
    fmt.Println(user.Email.String)
}
```

The `Valid` field indicates whether the value is non-NULL.

Use context cancellation to limit query execution time.

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

rows, err := db.QueryContext(ctx, "SELECT ...")
```

Cancelling the context interrupts long-running queries, preventing a single slow query from blocking a connection indefinitely.

## Common Pitfalls
1. **String concatenation in queries** — Even "internal" inputs can contain unexpected characters. Always use parameterized queries for every variable.
2. **Ignoring NULL handling** — Scanning a SQL NULL into a plain `string` causes a runtime error. Use `sql.NullString` or pointer types.

## Best Practices
1. **Use `QueryContext` with timeouts for all queries** — This prevents runaway queries from exhausting your connection pool.
2. **Audit for string concatenation regularly** — Use linters or code review to catch any SQL built with `+` or `fmt.Sprintf`.

## Summary
- Always use parameterized queries (`$1`, `?`) — never concatenate user input into SQL.
- Handle NULL values with `sql.NullString` and similar types.
- Use `context.WithTimeout` to limit query execution time and prevent resource exhaustion.

## Code Examples

**Two safe query patterns — parameterized queries and prepared statements both prevent SQL injection by separating query structure from values**

```go
// Safe: parameterized query
rows, err := db.Query(
    "SELECT * FROM users WHERE name = $1 AND status = $2",
    userName,
    "active",
)

// Also safe: prepared statement
stmt, _ := db.Prepare("SELECT * FROM users WHERE id = $1")
row := stmt.QueryRow(userID)
```


## Resources

- [database/sql Package](https://pkg.go.dev/database/sql) — Official Go documentation for parameterized queries and NULL handling
- [Accessing Databases](https://go.dev/doc/database/) — Official Go guide covering SQL security best practices

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*