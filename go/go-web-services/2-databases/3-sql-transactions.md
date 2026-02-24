---
source_course: "go-web-services"
source_lesson: "go-web-services-sql-transactions"
---

# Transactions

## Introduction
Transactions ensure that a group of database operations either all succeed or all fail together. In Go, the `database/sql` package provides `db.Begin()` to start a transaction and methods on the `*sql.Tx` object to execute queries within it.

## Key Concepts
- **db.Begin()**: Starts a new transaction and returns a `*sql.Tx` object bound to a single database connection.
- **tx.Rollback()**: Undoes all changes made within the transaction. Safe to call after `Commit()` (becomes a no-op).
- **tx.Commit()**: Finalizes the transaction, making all changes permanent.
- **tx.Exec() / tx.Query()**: Execute queries within the transaction — using `db` directly would bypass the transaction.

## Real World Context
Any operation involving multiple related writes needs transactions. Transferring money between accounts, creating an order with line items, or updating a user and their profile atomically all require transactions to prevent partial updates that corrupt data.

## Deep Dive

The standard transaction pattern in Go uses `defer tx.Rollback()` for safety — it is a no-op if the transaction was already committed.

```go
tx, err := db.Begin()
if err != nil {
    return err
}
defer tx.Rollback()

_, err = tx.Exec("UPDATE accounts SET balance = balance - $1 WHERE id = $2", 100, fromID)
if err != nil {
    return err
}

_, err = tx.Exec("UPDATE accounts SET balance = balance + $1 WHERE id = $2", 100, toID)
if err != nil {
    return err
}

return tx.Commit()
```

If any `Exec` fails and the function returns early, `defer tx.Rollback()` automatically undoes all changes.

For more control, use `BeginTx` with context and isolation options.

```go
tx, err := db.BeginTx(ctx, &sql.TxOptions{
    Isolation: sql.LevelSerializable,
    ReadOnly:  false,
})
```

Serializable isolation prevents all concurrent anomalies but reduces throughput.

A reusable transaction helper reduces boilerplate across your codebase.

```go
func withTx(db *sql.DB, fn func(*sql.Tx) error) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()

    if err := fn(tx); err != nil {
        return err
    }
    return tx.Commit()
}
```

This helper ensures every transaction follows the correct Begin/Rollback/Commit pattern.

## Common Pitfalls
1. **Using `db` instead of `tx` inside a transaction** — Queries on `db` use a different connection from the pool and are NOT part of the transaction.
2. **Forgetting `defer tx.Rollback()`** — Without it, an early return on error leaves the transaction open, holding a connection indefinitely.

## Best Practices
1. **Always `defer tx.Rollback()` immediately after `db.Begin()`** — This guarantees cleanup regardless of how the function exits.
2. **Create a `withTx` helper** — Centralizing the transaction pattern eliminates repetitive boilerplate and prevents mistakes.

## Summary
- Start transactions with `db.Begin()`, execute queries on the `tx` object, and finish with `tx.Commit()`.
- Always `defer tx.Rollback()` right after `Begin()` for automatic cleanup.
- Never use `db` directly inside a transaction — only use the `tx` object.

## Code Examples

**A bank transfer using a transaction — defer tx.Rollback() ensures atomicity even if an error occurs between the two updates**

```go
func transfer(db *sql.DB, from, to int, amount int) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()

    _, err = tx.Exec("UPDATE accounts SET balance = balance - $1 WHERE id = $2", amount, from)
    if err != nil {
        return err
    }

    _, err = tx.Exec("UPDATE accounts SET balance = balance + $1 WHERE id = $2", amount, to)
    if err != nil {
        return err
    }

    return tx.Commit()
}
```


## Resources

- [database/sql Package](https://pkg.go.dev/database/sql) — Official Go documentation for transaction methods Begin, Commit, and Rollback
- [Accessing Databases](https://go.dev/doc/database/) — Official Go guide covering transaction patterns

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*