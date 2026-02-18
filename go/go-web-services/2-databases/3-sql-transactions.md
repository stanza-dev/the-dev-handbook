---
source_course: "go-web-services"
source_lesson: "go-web-services-sql-transactions"
---

# Transactions

To ensure atomicity (all or nothing), use transactions.

```go
tx, err := db.Begin()
if err != nil {
    return err
}
defer tx.Rollback()  // Safe: no-op if committed

// Use tx, NOT db, for queries
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

## Context-Aware Transactions

```go
tx, err := db.BeginTx(ctx, &sql.TxOptions{
    Isolation: sql.LevelSerializable,
    ReadOnly:  false,
})
```

## Transaction Helper

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

## Code Examples

**Transaction Pattern**

```go
func transfer(db *sql.DB, from, to int, amount int) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()  // Safe no-op if committed

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


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*