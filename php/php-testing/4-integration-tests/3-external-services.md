---
source_course: "php-testing"
source_lesson: "php-testing-external-services"
---

# Testing External Services

## Introduction
Modern applications depend on external services — payment gateways, email providers, third-party APIs, and cloud storage. Testing code that interacts with these services requires strategies to avoid hitting real endpoints during tests, while still verifying that your code sends the right requests and handles the full range of responses.

## Key Concepts
- **HTTP Client Mocking**: Replacing the HTTP client with a stub that returns pre-defined responses instead of making real network requests.
- **Contract Testing**: Verifying that your code's requests match the external API's expected format, and that your code correctly handles the API's documented response shapes.
- **Response Recording/Replaying**: Capturing real API responses once and replaying them in tests, ensuring your tests use realistic data without network calls.
- **Circuit Breaker Pattern**: A resilience pattern where your code stops calling a failing service after repeated failures, which must be tested to verify correct behavior.

## Real World Context
Imagine your payment integration test charges a real credit card every time the test suite runs. Beyond the obvious cost, network latency makes the test slow, and transient failures make it flaky. By mocking the HTTP client, you get fast, deterministic tests that can simulate every response scenario — success, decline, timeout, and server error.

## Deep Dive
The most common approach is to stub the HTTP client interface. If your code depends on a `HttpClientInterface`, you can provide a stub that returns scripted responses.

Here is a stub-based approach for testing a payment gateway client:

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

interface HttpClientInterface
{
    public function post(string $url, array $options = []): ResponseInterface;
    public function get(string $url, array $options = []): ResponseInterface;
}

class PaymentGatewayTest extends TestCase
{
    public function testChargeReturnsTransactionId(): void
    {
        // Create a stub HTTP client that returns a success response
        $httpClient = $this->createStub(HttpClientInterface::class);
        $httpClient->method('post')
            ->willReturn(new JsonResponse(200, [
                'transaction_id' => 'txn_abc123',
                'status' => 'succeeded',
                'amount' => 4999,
            ]));

        $gateway = new PaymentGateway($httpClient, apiKey: 'test_key');
        $result = $gateway->charge(amount: 49.99, currency: 'USD', token: 'tok_visa');

        $this->assertSame('txn_abc123', $result->transactionId);
        $this->assertTrue($result->isSuccessful());
    }

    public function testChargeHandlesDeclinedCard(): void
    {
        $httpClient = $this->createStub(HttpClientInterface::class);
        $httpClient->method('post')
            ->willReturn(new JsonResponse(402, [
                'error' => [
                    'type' => 'card_error',
                    'message' => 'Your card was declined.',
                    'code' => 'card_declined',
                ],
            ]));

        $gateway = new PaymentGateway($httpClient, apiKey: 'test_key');
        $result = $gateway->charge(amount: 49.99, currency: 'USD', token: 'tok_declined');

        $this->assertFalse($result->isSuccessful());
        $this->assertSame('card_declined', $result->errorCode);
    }
}
```

The stub returns different responses for success and failure scenarios without making any network calls.

When you need to verify that your code sends the correct request body and headers, use a mock:

```php
<?php
public function testChargeSendsCorrectPayload(): void
{
    $httpClient = $this->createMock(HttpClientInterface::class);

    $httpClient->expects($this->once())
        ->method('post')
        ->with(
            $this->equalTo('https://api.payment.com/v1/charges'),
            $this->callback(function (array $options): bool {
                $body = $options['json'] ?? [];
                return $body['amount'] === 4999
                    && $body['currency'] === 'usd'
                    && isset($body['source'])
                    && str_starts_with($options['headers']['Authorization'] ?? '', 'Bearer ');
            })
        )
        ->willReturn(new JsonResponse(200, ['transaction_id' => 'txn_test']));

    $gateway = new PaymentGateway($httpClient, apiKey: 'test_key');
    $gateway->charge(amount: 49.99, currency: 'USD', token: 'tok_visa');
}
```

The `callback()` constraint verifies that the amount is sent in cents, the currency is lowercase, and the authorization header is present.

For response recording and replaying, you can use a simple file-based approach:

```php
<?php
class RecordedResponseClient implements HttpClientInterface
{
    public function __construct(
        private readonly string $fixturesDir
    ) {}

    public function post(string $url, array $options = []): ResponseInterface
    {
        return $this->loadFixture($url, 'POST');
    }

    public function get(string $url, array $options = []): ResponseInterface
    {
        return $this->loadFixture($url, 'GET');
    }

    private function loadFixture(string $url, string $method): ResponseInterface
    {
        $filename = md5($method . ':' . $url) . '.json';
        $path = $this->fixturesDir . '/' . $filename;

        if (!file_exists($path)) {
            throw new \RuntimeException(
                "No recorded response for {$method} {$url}. "
                . "Run tests in record mode first."
            );
        }

        $data = json_decode(file_get_contents($path), true);
        return new JsonResponse($data['status'], $data['body']);
    }
}
```

This approach uses fixture files containing recorded API responses. You record them once from the real API and replay them in all subsequent test runs.

To test timeout and network failure handling:

```php
<?php
public function testHandlesTimeoutGracefully(): void
{
    $httpClient = $this->createStub(HttpClientInterface::class);
    $httpClient->method('post')
        ->willThrowException(new ConnectionTimeoutException('Request timed out after 5s'));

    $gateway = new PaymentGateway($httpClient, apiKey: 'test_key');
    $result = $gateway->charge(amount: 49.99, currency: 'USD', token: 'tok_visa');

    $this->assertFalse($result->isSuccessful());
    $this->assertSame('timeout', $result->errorCode);
}
```

This verifies that your code degrades gracefully when the external service is unreachable.

## Common Pitfalls
1. **Testing against real external services in CI** — Real API calls in CI are slow, flaky, and can incur costs. Always stub or mock external HTTP clients in automated tests. Reserve real API calls for manual smoke tests or a dedicated integration test suite that runs separately.
2. **Not testing error responses** — Many developers only test the happy path (200 OK). External services return 4xx and 5xx responses, timeouts, and malformed data. Your tests must cover all these scenarios to ensure your error handling works.

## Best Practices
1. **Wrap external APIs in your own interface** — Create a `PaymentGatewayInterface` that your code depends on. The real implementation calls the API; the test double implements the same interface. This decouples your business logic from the specific API client library.
2. **Store response fixtures alongside tests** — Keep recorded API responses in a `fixtures/` directory next to your test files. Name them descriptively (e.g., `charge-success.json`, `charge-declined.json`) so other developers understand what each fixture represents.

## Summary
- Stub the HTTP client interface to avoid real network calls in tests.
- Use mocks with `callback()` constraints to verify that requests contain the correct payload and headers.
- Test error scenarios (declined cards, timeouts, server errors) in addition to success paths.
- Consider recording and replaying real API responses for realistic test data.
- Always wrap third-party APIs behind your own interface for maximum testability.

## Code Examples

**Testing an external weather API integration with stubbed success and error responses, showing graceful degradation**

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

interface WeatherApiInterface
{
    public function getForecast(string $city): array;
}

class WeatherServiceTest extends TestCase
{
    public function testGetForecastReturnsParsedData(): void
    {
        $api = $this->createStub(WeatherApiInterface::class);
        $api->method('getForecast')
            ->willReturn([
                'city' => 'Paris',
                'temperature' => 22.5,
                'condition' => 'sunny',
                'humidity' => 45,
            ]);

        $service = new WeatherService($api);
        $forecast = $service->getSummary('Paris');

        $this->assertSame('Paris', $forecast->city);
        $this->assertSame('sunny', $forecast->condition);
        $this->assertStringContainsString('22.5', $forecast->description);
    }

    public function testGetForecastHandlesApiError(): void
    {
        $api = $this->createStub(WeatherApiInterface::class);
        $api->method('getForecast')
            ->willThrowException(new \RuntimeException('API rate limit exceeded'));

        $service = new WeatherService($api);
        $forecast = $service->getSummary('Paris');

        // Service returns a fallback response instead of crashing
        $this->assertSame('unavailable', $forecast->condition);
        $this->assertStringContainsString('temporarily unavailable', $forecast->description);
    }
}
?>
```


## Resources

- [PHPUnit Test Doubles Documentation](https://docs.phpunit.de/en/12.0/test-doubles.html) — Official PHPUnit 12 documentation on creating stubs and mocks for HTTP client interfaces
- [PSR-18 HTTP Client Interface](https://www.php-fig.org/psr/psr-18/) — The PSR-18 standard for HTTP client interfaces, commonly used as the contract for mocking

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*