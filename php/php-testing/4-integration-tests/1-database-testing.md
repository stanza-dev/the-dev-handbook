---
source_course: "php-testing"
source_lesson: "php-testing-database-testing"
---

# Database Integration Tests

Integration tests verify that components work together correctly, including real database interactions.

## In-Memory SQLite

```php
<?php
class UserRepositoryTest extends TestCase
{
    private PDO $pdo;
    private UserRepository $repository;
    
    protected function setUp(): void
    {
        // Use in-memory SQLite for speed
        $this->pdo = new PDO('sqlite::memory:');
        $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        
        // Create schema
        $this->pdo->exec('
            CREATE TABLE users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                email TEXT UNIQUE NOT NULL,
                name TEXT NOT NULL,
                created_at TEXT DEFAULT CURRENT_TIMESTAMP
            )
        ');
        
        $this->repository = new UserRepository($this->pdo);
    }
    
    public function testCreateAndFindUser(): void
    {
        // Create user
        $user = $this->repository->create([
            'email' => 'john@example.com',
            'name' => 'John Doe',
        ]);
        
        $this->assertGreaterThan(0, $user->id);
        
        // Find user
        $found = $this->repository->find($user->id);
        
        $this->assertNotNull($found);
        $this->assertEquals('john@example.com', $found->email);
        $this->assertEquals('John Doe', $found->name);
    }
    
    public function testFindByEmail(): void
    {
        $this->repository->create([
            'email' => 'jane@example.com',
            'name' => 'Jane',
        ]);
        
        $user = $this->repository->findByEmail('jane@example.com');
        
        $this->assertNotNull($user);
        $this->assertEquals('Jane', $user->name);
    }
    
    public function testUniqueEmailConstraint(): void
    {
        $this->repository->create(['email' => 'test@example.com', 'name' => 'Test']);
        
        $this->expectException(PDOException::class);
        $this->repository->create(['email' => 'test@example.com', 'name' => 'Duplicate']);
    }
}
```

## Database Transactions for Isolation

```php
<?php
class TransactionalTestCase extends TestCase
{
    protected PDO $pdo;
    
    protected function setUp(): void
    {
        $this->pdo = new PDO($_ENV['DATABASE_URL']);
        $this->pdo->beginTransaction();
    }
    
    protected function tearDown(): void
    {
        // Roll back all changes after each test
        $this->pdo->rollBack();
    }
}

class OrderIntegrationTest extends TransactionalTestCase
{
    public function testCreateOrder(): void
    {
        // Changes are automatically rolled back
        $order = $this->createOrder();
        $this->assertNotNull($order->id);
    }
    
    public function testAnotherTest(): void
    {
        // Database is clean - no order from previous test
    }
}
```

## Fixtures and Seeders

```php
<?php
trait DatabaseFixtures
{
    protected function seedUsers(): array
    {
        $users = [];
        $stmt = $this->pdo->prepare(
            'INSERT INTO users (email, name) VALUES (:email, :name)'
        );
        
        foreach ($this->userFixtures() as $data) {
            $stmt->execute($data);
            $users[] = ['id' => $this->pdo->lastInsertId(), ...$data];
        }
        
        return $users;
    }
    
    private function userFixtures(): array
    {
        return [
            ['email' => 'admin@example.com', 'name' => 'Admin User'],
            ['email' => 'user@example.com', 'name' => 'Regular User'],
            ['email' => 'guest@example.com', 'name' => 'Guest User'],
        ];
    }
}

class UserSearchTest extends TestCase
{
    use DatabaseFixtures;
    
    public function testSearchUsers(): void
    {
        $users = $this->seedUsers();
        
        $results = $this->repository->search('User');
        
        $this->assertCount(2, $results);  // Admin User, Regular User
    }
}
```

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*