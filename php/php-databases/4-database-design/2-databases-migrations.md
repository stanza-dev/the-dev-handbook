---
source_course: "php-databases"
source_lesson: "php-databases-migrations"
---

# Database Migrations

Migrations version control your database schema, enabling reliable deployments.

## Simple Migration System

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

## Migration Runner

```php
<?php
class MigrationRunner {
    public function __construct(
        private PDO $pdo
    ) {
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
        $executed = $this->getExecutedMigrations();
        $batch = $this->getNextBatch();
        
        foreach ($migrations as $name => $migration) {
            if (in_array($name, $executed, true)) {
                continue;
            }
            
            echo "Running: $name\n";
            
            $this->pdo->beginTransaction();
            try {
                $migration->up($this->pdo);
                $this->recordMigration($name, $batch);
                $this->pdo->commit();
                echo "Completed: $name\n";
            } catch (Exception $e) {
                $this->pdo->rollBack();
                throw $e;
            }
        }
    }
    
    public function rollback(): void {
        $lastBatch = $this->getLastBatch();
        $migrations = $this->getMigrationsByBatch($lastBatch);
        
        foreach (array_reverse($migrations) as $migration) {
            echo "Rolling back: {$migration['migration']}\n";
            // Load and run down() method
        }
    }
    
    private function getExecutedMigrations(): array {
        return $this->pdo
            ->query('SELECT migration FROM migrations')
            ->fetchAll(PDO::FETCH_COLUMN);
    }
    
    private function getNextBatch(): int {
        return (int) $this->pdo
            ->query('SELECT COALESCE(MAX(batch), 0) + 1 FROM migrations')
            ->fetchColumn();
    }
    
    private function recordMigration(string $name, int $batch): void {
        $stmt = $this->pdo->prepare(
            'INSERT INTO migrations (migration, batch) VALUES (:name, :batch)'
        );
        $stmt->execute(['name' => $name, 'batch' => $batch]);
    }
}
```

## Column Modifications

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

class AddIndexToOrdersStatus extends Migration {
    public function up(PDO $pdo): void {
        $pdo->exec('CREATE INDEX idx_orders_status ON orders(status)');
    }
    
    public function down(PDO $pdo): void {
        $pdo->exec('DROP INDEX idx_orders_status ON orders');
    }
}
```

## Code Examples

**Complete e-commerce schema migration**

```php
<?php
declare(strict_types=1);

// Example migration for an e-commerce schema
class CreateEcommerceSchema extends Migration {
    public function up(PDO $pdo): void {
        // Categories
        $pdo->exec('
            CREATE TABLE categories (
                id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                parent_id INT UNSIGNED NULL,
                name VARCHAR(255) NOT NULL,
                slug VARCHAR(255) NOT NULL UNIQUE,
                description TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                FOREIGN KEY (parent_id) REFERENCES categories(id) ON DELETE SET NULL
            )
        ');
        
        // Products
        $pdo->exec('
            CREATE TABLE products (
                id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                category_id INT UNSIGNED,
                sku VARCHAR(50) NOT NULL UNIQUE,
                name VARCHAR(255) NOT NULL,
                slug VARCHAR(255) NOT NULL UNIQUE,
                description TEXT,
                price DECIMAL(10, 2) NOT NULL,
                stock_quantity INT UNSIGNED DEFAULT 0,
                status ENUM("draft", "active", "discontinued") DEFAULT "draft",
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE SET NULL,
                INDEX idx_status (status),
                INDEX idx_category (category_id),
                FULLTEXT INDEX idx_search (name, description)
            )
        ');
        
        // Orders
        $pdo->exec('
            CREATE TABLE orders (
                id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                user_id INT UNSIGNED NOT NULL,
                status ENUM("pending", "paid", "shipped", "delivered", "cancelled") DEFAULT "pending",
                subtotal DECIMAL(10, 2) NOT NULL,
                tax DECIMAL(10, 2) DEFAULT 0,
                shipping DECIMAL(10, 2) DEFAULT 0,
                total DECIMAL(10, 2) NOT NULL,
                notes TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                FOREIGN KEY (user_id) REFERENCES users(id),
                INDEX idx_user (user_id),
                INDEX idx_status (status),
                INDEX idx_created (created_at)
            )
        ');
        
        // Order Items
        $pdo->exec('
            CREATE TABLE order_items (
                id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                order_id INT UNSIGNED NOT NULL,
                product_id INT UNSIGNED NOT NULL,
                product_name VARCHAR(255) NOT NULL,
                quantity INT UNSIGNED NOT NULL,
                unit_price DECIMAL(10, 2) NOT NULL,
                total_price DECIMAL(10, 2) NOT NULL,
                FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
                FOREIGN KEY (product_id) REFERENCES products(id),
                INDEX idx_order (order_id)
            )
        ');
    }
    
    public function down(PDO $pdo): void {
        $pdo->exec('DROP TABLE IF EXISTS order_items');
        $pdo->exec('DROP TABLE IF EXISTS orders');
        $pdo->exec('DROP TABLE IF EXISTS products');
        $pdo->exec('DROP TABLE IF EXISTS categories');
    }
}
?>
```


---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*