---
source_course: "php-testing"
source_lesson: "php-testing-setup-teardown"
---

# Setup and Teardown

PHPUnit provides hooks to prepare and clean up test environments.

## Instance Methods

```php
<?php
class UserServiceTest extends TestCase
{
    private UserService $service;
    private PDO $pdo;
    
    // Runs BEFORE each test method
    protected function setUp(): void
    {
        parent::setUp();
        
        $this->pdo = new PDO('sqlite::memory:');
        $this->pdo->exec('CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)');
        
        $this->service = new UserService($this->pdo);
    }
    
    // Runs AFTER each test method
    protected function tearDown(): void
    {
        $this->pdo = null;  // Close connection
        
        parent::tearDown();
    }
    
    public function testCreateUser(): void
    {
        // Fresh database for each test
        $user = $this->service->create('John');
        $this->assertEquals('John', $user->name);
    }
    
    public function testAnotherTest(): void
    {
        // Also has fresh database
        $count = $this->service->count();
        $this->assertEquals(0, $count);
    }
}
```

## Static Methods (Once Per Class)

```php
<?php
class ExpensiveSetupTest extends TestCase
{
    private static PDO $sharedPdo;
    
    // Runs ONCE before any test in this class
    public static function setUpBeforeClass(): void
    {
        parent::setUpBeforeClass();
        
        self::$sharedPdo = new PDO('sqlite::memory:');
        self::$sharedPdo->exec('CREATE TABLE config (key TEXT, value TEXT)');
        self::$sharedPdo->exec("INSERT INTO config VALUES ('version', '1.0')");
    }
    
    // Runs ONCE after all tests in this class
    public static function tearDownAfterClass(): void
    {
        self::$sharedPdo = null;
        
        parent::tearDownAfterClass();
    }
    
    public function testReadConfig(): void
    {
        $stmt = self::$sharedPdo->query('SELECT value FROM config WHERE key = "version"');
        $this->assertEquals('1.0', $stmt->fetchColumn());
    }
}
```

## Execution Order

```
setUpBeforeClass()  // Once per class
    setUp()         // Before test 1
        test1()
    tearDown()      // After test 1
    setUp()         // Before test 2
        test2()
    tearDown()      // After test 2
tearDownAfterClass()  // Once per class
```

## Test Dependencies

```php
<?php
class OrderTest extends TestCase
{
    public function testCreateOrder(): int
    {
        $order = new Order();
        $order->addItem('Widget', 10.00);
        $order->save();
        
        $this->assertGreaterThan(0, $order->id);
        
        return $order->id;  // Pass to dependent test
    }
    
    /**
     * @depends testCreateOrder
     */
    public function testOrderCanBePaid(int $orderId): void
    {
        $order = Order::find($orderId);
        $order->pay();
        
        $this->assertEquals('paid', $order->status);
    }
}
```

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*