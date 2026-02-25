---
source_course: "php-testing"
source_lesson: "php-testing-api-testing"
---

# Testing HTTP Endpoints

## Introduction
HTTP endpoint tests verify that your application responds correctly to real HTTP requests — returning the right status codes, headers, and response bodies. These tests sit one layer above unit tests, exercising the full request/response cycle including routing, middleware, validation, and serialization.

## Key Concepts
- **HTTP Test Client**: A component that sends HTTP requests to your application in-process without needing a running web server.
- **Status Code Assertions**: Verifying that the response has the expected HTTP status (200, 201, 404, 422, etc.).
- **JSON Assertions**: Checking the structure and values of JSON response bodies.
- **Request Factories**: Helper methods that construct HTTP requests with specific headers, query parameters, and body payloads.

## Real World Context
Unit testing a controller with mocked request/response objects only tests the controller logic. It misses routing configuration, middleware execution order, content negotiation, and serialization bugs. HTTP endpoint tests catch all of these because they exercise the full stack up to the framework's HTTP kernel.

## Deep Dive
PHP frameworks like Symfony and Laravel provide built-in HTTP testing clients. For framework-agnostic code or custom applications, you can build a lightweight test client using PHP's built-in server or by directly invoking your application's request handler.

Here is a pattern for testing a JSON API endpoint using a test client abstraction:

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class UserApiTest extends TestCase
{
    private TestHttpClient $client;

    protected function setUp(): void
    {
        parent::setUp();
        $this->client = new TestHttpClient(createApp());
    }

    public function testListUsersReturns200(): void
    {
        $response = $this->client->get('/api/users');

        $this->assertSame(200, $response->getStatusCode());
        $this->assertSame('application/json', $response->getHeaderLine('Content-Type'));
    }

    public function testListUsersReturnsJsonArray(): void
    {
        $response = $this->client->get('/api/users');
        $body = json_decode($response->getBody(), true);

        $this->assertIsArray($body);
        $this->assertArrayHasKey('data', $body);
    }
}
```

The test client wraps the application and exposes HTTP methods (`get()`, `post()`, etc.) that return response objects. No real server is running — requests are handled in-process.

Testing POST endpoints with JSON payloads follows the same pattern:

```php
<?php
public function testCreateUserReturns201(): void
{
    $response = $this->client->post('/api/users', [
        'json' => [
            'email' => 'newuser@example.com',
            'name' => 'New User',
        ],
    ]);

    $this->assertSame(201, $response->getStatusCode());

    $body = json_decode($response->getBody(), true);
    $this->assertSame('newuser@example.com', $body['data']['email']);
    $this->assertArrayHasKey('id', $body['data']);
}

public function testCreateUserWithInvalidEmailReturns422(): void
{
    $response = $this->client->post('/api/users', [
        'json' => [
            'email' => 'not-an-email',
            'name' => 'Bad User',
        ],
    ]);

    $this->assertSame(422, $response->getStatusCode());

    $body = json_decode($response->getBody(), true);
    $this->assertArrayHasKey('errors', $body);
    $this->assertArrayHasKey('email', $body['errors']);
}
```

Note how the validation test verifies both the status code (422 Unprocessable Entity) and the error response structure. This ensures the API communicates validation failures properly to clients.

For endpoints that require authentication, pass headers with the request:

```php
<?php
public function testProtectedEndpointRequiresAuth(): void
{
    // No auth header — should be rejected
    $response = $this->client->get('/api/admin/dashboard');
    $this->assertSame(401, $response->getStatusCode());
}

public function testProtectedEndpointWithValidToken(): void
{
    $token = $this->createTestToken(userId: 1, role: 'admin');

    $response = $this->client->get('/api/admin/dashboard', [
        'headers' => [
            'Authorization' => 'Bearer ' . $token,
        ],
    ]);

    $this->assertSame(200, $response->getStatusCode());
}
```

These tests verify that your authentication middleware correctly rejects unauthenticated requests and accepts valid tokens.

A helper method for asserting JSON response structure keeps tests readable:

```php
<?php
private function assertJsonResponse(
    object $response,
    int $expectedStatus,
    array $expectedKeys = []
): array {
    $this->assertSame($expectedStatus, $response->getStatusCode());

    $body = json_decode($response->getBody(), true);
    $this->assertIsArray($body, 'Response body is not valid JSON');

    foreach ($expectedKeys as $key) {
        $this->assertArrayHasKey($key, $body, "Missing key: $key");
    }

    return $body;
}
```

This utility reduces repetition across your API tests and produces clear failure messages when assertions fail.

## Common Pitfalls
1. **Testing only the happy path** — API tests must cover error cases: invalid input (422), missing resources (404), unauthorized access (401/403), and server errors. Clients depend on correct error responses for their own error handling.
2. **Hardcoding IDs in URLs** — Tests that use `/api/users/1` assume a specific auto-increment value. Instead, create the resource first and extract its ID from the response to build subsequent request URLs.

## Best Practices
1. **Assert both status code and body** — A 200 status alone does not prove correctness. Always verify the response body structure and key values to ensure the endpoint returns the expected data.
2. **Organize tests by endpoint** — Group test methods by the API endpoint they exercise (e.g., `testCreateUser*`, `testGetUser*`, `testDeleteUser*`). This makes it easy to find and maintain tests as the API evolves.

## Summary
- HTTP endpoint tests exercise the full request/response cycle including routing, middleware, and serialization.
- Use a test HTTP client to send requests in-process without starting a real server.
- Assert status codes, response headers, and JSON body structure for every endpoint.
- Cover error cases (422, 404, 401) in addition to happy-path success responses.
- Extract helper methods for common assertion patterns to keep tests readable.

## Code Examples

**API endpoint tests covering create, retrieve, not-found, and validation error scenarios with JSON assertions**

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class ProductApiTest extends TestCase
{
    private TestHttpClient $client;

    protected function setUp(): void
    {
        $this->client = new TestHttpClient(createApp());
    }

    public function testGetProductReturns200WithCorrectStructure(): void
    {
        // First create a product so we have a known ID
        $createResponse = $this->client->post('/api/products', [
            'json' => [
                'name' => 'Wireless Mouse',
                'price' => 29.99,
                'category' => 'electronics',
            ],
        ]);
        $productId = json_decode($createResponse->getBody(), true)['data']['id'];

        // Now fetch the product by its ID
        $response = $this->client->get("/api/products/{$productId}");

        $this->assertSame(200, $response->getStatusCode());

        $body = json_decode($response->getBody(), true);
        $this->assertSame('Wireless Mouse', $body['data']['name']);
        $this->assertSame(29.99, $body['data']['price']);
        $this->assertSame('electronics', $body['data']['category']);
    }

    public function testGetNonExistentProductReturns404(): void
    {
        $response = $this->client->get('/api/products/999999');

        $this->assertSame(404, $response->getStatusCode());

        $body = json_decode($response->getBody(), true);
        $this->assertSame('Product not found', $body['error']['message']);
    }

    public function testCreateProductWithMissingFieldsReturns422(): void
    {
        $response = $this->client->post('/api/products', [
            'json' => ['name' => 'Incomplete Product'],
        ]);

        $this->assertSame(422, $response->getStatusCode());

        $errors = json_decode($response->getBody(), true)['errors'];
        $this->assertArrayHasKey('price', $errors);
    }
}
?>
```


## Resources

- [PHPUnit Documentation](https://docs.phpunit.de/en/12.0/writing-tests-for-phpunit.html) — Official PHPUnit 12 documentation on writing tests
- [PSR-7 HTTP Message Interfaces](https://www.php-fig.org/psr/psr-7/) — The PSR-7 standard for HTTP message interfaces used in PHP HTTP testing

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*