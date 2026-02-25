---
source_course: "php-databases"
source_lesson: "php-databases-migrations"
---

# Database Migrations

## Introduction

Database migrations version-control your schema, enabling reproducible deployments, safe rollbacks, and team collaboration. Instead of applying SQL changes manually, migrations define each change as code that can be tracked in git.

## Key Concepts

- **Migration**: A class with `up()` (apply change) and `down()` (revert change) methods that modify the database schema.
- **Batch**: A group of migrations run together. Rollback reverts the latest batch, not individual migrations.
- **Migrations Table**: A metadata table that records which migrations have been applied and in which batch.

## Real World Context

Without migrations, deploying a schema change means emailing SQL scripts, hoping they run in order, and praying nobody forgets a step. Migrations make schema changes as reliable and reviewable as code changes.

## Deep Dive

A migration is an abstract class with `up()` and `down()` methods:

```php
<?php
abstract class Migration {
    abstract public function up(PDO $pdo): void;
    abstract public function down(PDO $pdo): void;
}

class CreateUsersTable extends Migration {
    public function up(PDO $pdo): void {
        $pdo->exec('
            CREATE TABLE users (
                id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                name VARCHAR(255) NOT NULL,
                email VARCHAR(255) NOT NULL UNIQUE,
                password_hash VARCHAR(255) NOT NULL,
                status ENUM("active", "inactive", "banned") DEFAULT "active",
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                INDEX idx_status (status),
                INDEX idx_created (created_at)
            ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
        ');
    }
    
    public function down(PDO $pdo): void {
        $pdo->exec('DROP TABLE IF EXISTS users');
    }
}
```

The migration runner tracks which migrations have been applied:

```php
<?php
class MigrationRunner {
    public function __construct(private PDO $pdo) {
        $this->createMigrationsTable();
    }
    
    private function createMigrationsTable(): void {
        $this->pdo->exec('
            CREATE TABLE IF NOT EXISTS migrations (
                id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                migration VARCHAR(255) NOT NULL,
                batch INT UNSIGNED NOT NULL,
                executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ');
    }
    
    public function run(array $migrations): void {
        $executed = $this->getExecuted();
        $batch = $this->getNextBatch();
        
        foreach ($migrations as $name => $migration) {
            if (in_array($name, $executed, true)) {
                continue;
            }
            $this->pdo->beginTransaction();
            try {
                $migration->up($this->pdo);
                $this->record($name, $batch);
                $this->pdo->commit();
            } catch (Exception $e) {
                $this->pdo->rollBack();
                throw $e;
            }
        }
    }
    
    private function getExecuted(): array {
        return $this->pdo
            ->query('SELECT migration FROM migrations')
            ->fetchAll(PDO::FETCH_COLUMN);
    }
    
    private function getNextBatch(): int {
        return (int) $this->pdo
            ->query('SELECT COALESCE(MAX(batch), 0) + 1 FROM migrations')
            ->fetchColumn();
    }
    
    private function record(string $name, int $batch): void {
        $stmt = $this->pdo->prepare(
            'INSERT INTO migrations (migration, batch) VALUES (:name, :batch)'
        );
        $stmt->execute(['name' => $name, 'batch' => $batch]);
    }
}
```

Column modifications are also migrations:

```php
<?php
class AddAvatarToUsers extends Migration {
    public function up(PDO $pdo): void {
        $pdo->exec('ALTER TABLE users ADD COLUMN avatar_url VARCHAR(500) NULL AFTER email');
    }
    
    public function down(PDO $pdo): void {
        $pdo->exec('ALTER TABLE users DROP COLUMN avatar_url');
    }
}
```

Each migration runs in its own transaction, so a failure in one migration does not corrupt previously applied migrations.

## Common Pitfalls

1. **Writing irreversible migrations without a down() method** — If you cannot write a meaningful `down()`, document it clearly. Some changes (like dropping a column with data) are inherently irreversible.
2. **Running migrations outside of transactions** — DDL statements in MySQL auto-commit, so wrapping them in a transaction has no effect. PostgreSQL does support transactional DDL.

## Best Practices

1. **Name migrations with timestamps** — Use `YYYY_MM_DD_HHMMSS_description` to ensure consistent ordering across team members.
2. **Never modify an already-executed migration** — If a migration has been deployed, create a new migration to fix or amend it. Modifying executed migrations causes drift between environments.

## Summary

- Migrations version-control database schema changes with reversible `up()` and `down()` methods.
- A migrations table tracks which changes have been applied and in which batch.
- Each migration runs in a transaction for safe rollback on failure.
- Never modify deployed migrations; create new ones to fix issues.

## Code Examples

**E-commerce schema migration with proper ordering**

```php
<?php
declare(strict_types=1);

class CreateEcommerceSchema extends Migration {
    public function up(PDO $pdo): void {
        $pdo->exec('
            CREATE TABLE categories (
                id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                parent_id INT UNSIGNED NULL,
                name VARCHAR(255) NOT NULL,
                slug VARCHAR(255) NOT NULL UNIQUE,
                FOREIGN KEY (parent_id) REFERENCES categories(id) ON DELETE SET NULL
            )
        ');
        $pdo->exec('
            CREATE TABLE products (
                id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                category_id INT UNSIGNED,
                sku VARCHAR(50) NOT NULL UNIQUE,
                name VARCHAR(255) NOT NULL,
                price DECIMAL(10, 2) NOT NULL,
                stock_quantity INT UNSIGNED DEFAULT 0,
                status ENUM("draft", "active", "discontinued") DEFAULT "draft",
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                FOREIGN KEY (category_id) REFERENCES categories(id),
                INDEX idx_status (status),
                FULLTEXT INDEX idx_search (name)
            )
        ');
    }
    
    public function down(PDO $pdo): void {
        $pdo->exec('DROP TABLE IF EXISTS products');
        $pdo->exec('DROP TABLE IF EXISTS categories');
    }
}
?>
```


## Resources

- [Database Migration Best Practices](https://www.prisma.io/dataguide/types/relational/what-are-database-migrations) — Overview of database migration concepts

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*