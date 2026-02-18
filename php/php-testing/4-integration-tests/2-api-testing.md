---
source_course: "php-testing"
source_lesson: "php-testing-api-testing"
---

# HTTP API Testing

Test your API endpoints to verify request handling and responses.

## Simple API Test

```php
<?php
class ApiTestCase extends TestCase
{
    protected function request(
        string $method,
        string $uri,
        array $data = [],
        array $headers = []
    ): array {
        // Simulate request environment
        $_SERVER['REQUEST_METHOD'] = $method;
        $_SERVER['REQUEST_URI'] = $uri;
        $_SERVER['CONTENT_TYPE'] = 'application/json';
        
        foreach ($headers as $name => $value) {
            $key = 'HTTP_' . strtoupper(str_replace('-', '_', $name));
            $_SERVER[$key] = $value;
        }
        
        // Simulate JSON body
        $GLOBALS['test_input'] = json_encode($data);
        
        // Capture output
        ob_start();
        
        try {
            require 'public/index.php';
        } catch (SystemExit $e) {
            // Caught exit() call
        }
        
        $output = ob_get_clean();
        
        return [
            'status' => http_response_code(),
            'body' => json_decode($output, true),
            'raw' => $output,
        ];
    }
    
    protected function get(string $uri, array $headers = []): array {
        return $this->request('GET', $uri, [], $headers);
    }
    
    protected function post(string $uri, array $data, array $headers = []): array {
        return $this->request('POST', $uri, $data, $headers);
    }
}
```

## Testing API Endpoints

```php
<?php
class UserApiTest extends ApiTestCase
{
    public function testListUsers(): void
    {
        $response = $this->get('/api/users');
        
        $this->assertEquals(200, $response['status']);
        $this->assertArrayHasKey('data', $response['body']);
        $this->assertIsArray($response['body']['data']);
    }
    
    public function testCreateUser(): void
    {
        $response = $this->post('/api/users', [
            'name' => 'John Doe',
            'email' => 'john@example.com',
        ]);
        
        $this->assertEquals(201, $response['status']);
        $this->assertEquals('John Doe', $response['body']['data']['name']);
    }
    
    public function testValidationError(): void
    {
        $response = $this->post('/api/users', [
            'name' => 'John',
            // Missing email
        ]);
        
        $this->assertEquals(422, $response['status']);
        $this->assertArrayHasKey('error', $response['body']);
        $this->assertArrayHasKey('details', $response['body']['error']);
        $this->assertArrayHasKey('email', $response['body']['error']['details']);
    }
    
    public function testAuthenticationRequired(): void
    {
        $response = $this->get('/api/profile');
        
        $this->assertEquals(401, $response['status']);
    }
    
    public function testAuthenticatedRequest(): void
    {
        $token = $this->getTestToken();
        
        $response = $this->get('/api/profile', [
            'Authorization' => "Bearer $token",
        ]);
        
        $this->assertEquals(200, $response['status']);
    }
}
```

## Code Examples

**Complete integration test for order workflow**

```php
<?php
declare(strict_types=1);

// Complete integration test suite
class OrderWorkflowTest extends TestCase
{
    private PDO $pdo;
    private OrderService $orderService;
    private PaymentService $paymentService;
    private InventoryService $inventoryService;
    
    protected function setUp(): void
    {
        // Real database connection
        $this->pdo = new PDO('sqlite::memory:');
        $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        
        $this->createSchema();
        $this->seedData();
        
        // Real services with real dependencies
        $productRepo = new ProductRepository($this->pdo);
        $orderRepo = new OrderRepository($this->pdo);
        
        // Only mock external services
        $paymentGateway = $this->createMock(PaymentGateway::class);
        $paymentGateway->method('charge')->willReturn('txn_123');
        
        $this->inventoryService = new InventoryService($this->pdo);
        $this->paymentService = new PaymentService($paymentGateway);
        $this->orderService = new OrderService(
            $orderRepo,
            $productRepo,
            $this->inventoryService,
            $this->paymentService
        );
    }
    
    private function createSchema(): void
    {
        $this->pdo->exec('
            CREATE TABLE products (
                id INTEGER PRIMARY KEY,
                name TEXT,
                price REAL,
                stock INTEGER
            );
            CREATE TABLE orders (
                id INTEGER PRIMARY KEY,
                status TEXT,
                total REAL
            );
            CREATE TABLE order_items (
                id INTEGER PRIMARY KEY,
                order_id INTEGER,
                product_id INTEGER,
                quantity INTEGER,
                price REAL
            );
        ');
    }
    
    private function seedData(): void
    {
        $this->pdo->exec("
            INSERT INTO products (id, name, price, stock) VALUES
            (1, 'Widget', 10.00, 100),
            (2, 'Gadget', 25.00, 50),
            (3, 'Thing', 5.00, 0)
        ");
    }
    
    public function testCompleteOrderWorkflow(): void
    {
        // Create order
        $order = $this->orderService->createOrder([
            ['product_id' => 1, 'quantity' => 2],  // 2 Widgets = $20
            ['product_id' => 2, 'quantity' => 1],  // 1 Gadget = $25
        ]);
        
        $this->assertEquals('pending', $order->status);
        $this->assertEquals(45.00, $order->total);
        
        // Check inventory was reserved
        $widget = $this->getProduct(1);
        $this->assertEquals(98, $widget['stock']);  // 100 - 2
        
        // Process payment
        $this->orderService->processPayment($order->id);
        
        $order = $this->orderService->find($order->id);
        $this->assertEquals('paid', $order->status);
    }
    
    public function testCannotOrderOutOfStockProduct(): void
    {
        $this->expectException(OutOfStockException::class);
        
        $this->orderService->createOrder([
            ['product_id' => 3, 'quantity' => 1],  // Thing has 0 stock
        ]);
    }
    
    public function testOrderCancellationRestoresStock(): void
    {
        $order = $this->orderService->createOrder([
            ['product_id' => 1, 'quantity' => 5],
        ]);
        
        $stockBefore = $this->getProduct(1)['stock'];  // 95
        
        $this->orderService->cancel($order->id);
        
        $stockAfter = $this->getProduct(1)['stock'];  // 100
        
        $this->assertEquals($stockBefore + 5, $stockAfter);
    }
    
    private function getProduct(int $id): array
    {
        $stmt = $this->pdo->prepare('SELECT * FROM products WHERE id = ?');
        $stmt->execute([$id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
}
?>
```


---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*