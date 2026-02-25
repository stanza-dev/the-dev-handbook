---
source_course: "php-testing"
source_lesson: "php-testing-database-testing"
---

# Database Testing Fundamentals

## Introduction
Unit tests with mocks verify logic in isolation, but they cannot catch bugs in SQL queries, schema mismatches, or transaction behavior. Database integration tests run real queries against a real (or in-memory) database to verify that your data layer works correctly end-to-end.

## Key Concepts
- **Test Database**: A dedicated database instance used only for tests, separate from development and production data.
- **Transaction Wrapping**: Running each test inside a database transaction that is rolled back afterward, ensuring every test starts with a clean slate.
- **Seeding**: Inserting known data before a test runs so assertions have predictable values to check against.
- **setUp/tearDown**: PHPUnit lifecycle hooks used to prepare and clean up the database state for each test.

## Real World Context
A stubbed repository always returns exactly what you tell it to. It cannot reveal that your `WHERE` clause has a typo, that a column was renamed in a migration, or that a `JOIN` produces duplicate rows. Database integration tests catch these real-world bugs that unit tests simply cannot see.

## Deep Dive
The simplest approach for PHP database testing uses SQLite's in-memory mode, which creates a fresh database that lives only in RAM and vanishes when the connection closes.

Here is a base test class that sets up an in-memory SQLite database:

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

abstract class DatabaseTestCase extends TestCase
{
    protected PDO $pdo;

    protected function setUp(): void
    {
        parent::setUp();

        // Create a fresh in-memory database for each test
        $this->pdo = new PDO('sqlite::memory:');
        $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

        // Run the schema setup
        $this->createTables();
    }

    protected function createTables(): void
    {
        $this->pdo->exec('
            CREATE TABLE users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                email TEXT NOT NULL UNIQUE,
                name TEXT NOT NULL,
                created_at TEXT DEFAULT CURRENT_TIMESTAMP
            )
        ');

        $this->pdo->exec('
            CREATE TABLE orders (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER NOT NULL,
                total REAL NOT NULL,
                status TEXT DEFAULT "pending",
                FOREIGN KEY (user_id) REFERENCES users(id)
            )
        ');
    }

    protected function seedUser(string $email, string $name): int
    {
        $stmt = $this->pdo->prepare('INSERT INTO users (email, name) VALUES (?, ?)');
        $stmt->execute([$email, $name]);
        return (int) $this->pdo->lastInsertId();
    }
}
```

This base class creates a fresh database with tables before every test. The `seedUser()` helper makes it easy to insert test data.

Now a concrete test that uses this base class:

```php
<?php
class UserRepositoryTest extends DatabaseTestCase
{
    private UserRepository $repository;

    protected function setUp(): void
    {
        parent::setUp();
        $this->repository = new UserRepository($this->pdo);
    }

    public function testFindByEmailReturnsUser(): void
    {
        // Seed: insert a known user
        $this->seedUser('alice@example.com', 'Alice');

        // Act: look up the user by email
        $user = $this->repository->findByEmail('alice@example.com');

        // Assert: verify the returned data
        $this->assertNotNull($user);
        $this->assertSame('Alice', $user['name']);
    }

    public function testFindByEmailReturnsNullForUnknownUser(): void
    {
        // No seeding — database is empty
        $result = $this->repository->findByEmail('nobody@example.com');

        $this->assertNull($result);
    }
}
```

Each test gets a fresh database, so there is no risk of leftover data from a previous test causing interference.

For applications using MySQL or PostgreSQL in production, you may need a real test database instead of SQLite. The transaction-wrapping pattern keeps the database clean between tests:

```php
<?php
abstract class TransactionalTestCase extends TestCase
{
    protected PDO $pdo;

    protected function setUp(): void
    {
        parent::setUp();

        // Connect to the test database (configured in phpunit.xml)
        $this->pdo = new PDO(
            $_ENV['TEST_DATABASE_DSN'],
            $_ENV['TEST_DATABASE_USER'],
            $_ENV['TEST_DATABASE_PASS']
        );
        $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

        // Start a transaction that will be rolled back after the test
        $this->pdo->beginTransaction();
    }

    protected function tearDown(): void
    {
        // Roll back all changes made during the test
        if ($this->pdo->inTransaction()) {
            $this->pdo->rollBack();
        }

        parent::tearDown();
    }
}
```

The transaction starts in `setUp()` and rolls back in `tearDown()`, so every `INSERT`, `UPDATE`, or `DELETE` executed during the test is undone automatically. This is faster than truncating tables and guarantees isolation.

## Common Pitfalls
1. **SQLite vs production database differences** — SQLite lacks features like `ENUM`, stored procedures, and some `JOIN` behaviors. If your queries use database-specific features, test against the same database engine you use in production.
2. **Shared test data between tests** — If tests share a database without transaction wrapping or table truncation, one test's data can leak into another, causing flaky failures that are difficult to debug.

## Best Practices
1. **Use transaction rollback for isolation** — Wrapping each test in a transaction that is rolled back is the fastest and most reliable way to isolate database tests. It avoids the overhead of recreating tables or truncating data.
2. **Keep seed data minimal** — Only insert the rows your test actually needs. Large seed datasets make tests slow and hard to understand. Use helper methods like `seedUser()` to insert precisely what each test requires.

## Summary
- Database integration tests verify SQL queries, schema correctness, and transaction behavior that unit tests cannot catch.
- Use SQLite in-memory databases for fast, isolated tests when your queries are database-agnostic.
- Wrap each test in a transaction and roll back in `tearDown()` to guarantee a clean slate.
- Create a base `DatabaseTestCase` class with schema setup and seeding helpers.
- Keep seed data minimal and specific to each test's requirements.

## Code Examples

**Complete database integration test using SQLite in-memory with create, retrieve, update, and query operations**

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class OrderRepositoryTest extends TestCase
{
    private PDO $pdo;
    private OrderRepository $repository;

    protected function setUp(): void
    {
        $this->pdo = new PDO('sqlite::memory:');
        $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

        $this->pdo->exec('
            CREATE TABLE orders (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                customer_email TEXT NOT NULL,
                total REAL NOT NULL,
                status TEXT DEFAULT "pending"
            )
        ');

        $this->repository = new OrderRepository($this->pdo);
    }

    public function testCreateAndRetrieveOrder(): void
    {
        $orderId = $this->repository->create(
            customerEmail: 'buyer@shop.com',
            total: 59.99
        );

        $order = $this->repository->findById($orderId);

        $this->assertSame('buyer@shop.com', $order['customer_email']);
        $this->assertSame(59.99, (float) $order['total']);
        $this->assertSame('pending', $order['status']);
    }

    public function testUpdateOrderStatus(): void
    {
        $orderId = $this->repository->create('buyer@shop.com', 25.00);

        $this->repository->updateStatus($orderId, 'shipped');

        $order = $this->repository->findById($orderId);
        $this->assertSame('shipped', $order['status']);
    }

    public function testFindByCustomerReturnsAllOrders(): void
    {
        $this->repository->create('vip@shop.com', 100.00);
        $this->repository->create('vip@shop.com', 200.00);
        $this->repository->create('other@shop.com', 50.00);

        $orders = $this->repository->findByCustomer('vip@shop.com');

        $this->assertCount(2, $orders);
    }
}
?>
```


## Resources

- [PHPUnit Database Testing](https://docs.phpunit.de/en/12.0/fixtures.html) — Official PHPUnit 12 documentation on test fixtures and setUp/tearDown lifecycle
- [PHP PDO Documentation](https://www.php.net/manual/en/book.pdo.php) — PHP manual for PDO, used in database integration tests

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*