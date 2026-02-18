---
source_course: "php-testing"
source_lesson: "php-testing-assertions"
---

# Assertions in Depth

Assertions are the heart of testing. They verify expected outcomes.

## Basic Assertions

```php
<?php
class AssertionExamplesTest extends TestCase
{
    public function testBasicAssertions(): void
    {
        // Equality
        $this->assertEquals('hello', 'hello');      // Loose comparison
        $this->assertSame('hello', 'hello');        // Strict (===)
        $this->assertNotEquals(1, 2);
        
        // Boolean
        $this->assertTrue(true);
        $this->assertFalse(false);
        
        // Null
        $this->assertNull(null);
        $this->assertNotNull('value');
        
        // Type
        $this->assertIsArray([1, 2, 3]);
        $this->assertIsString('text');
        $this->assertIsInt(42);
        $this->assertIsBool(true);
        $this->assertInstanceOf(User::class, new User());
    }
}
```

## Array Assertions

```php
<?php
public function testArrayAssertions(): void
{
    $array = ['name' => 'John', 'age' => 30, 'city' => 'NYC'];
    
    // Contains
    $this->assertArrayHasKey('name', $array);
    $this->assertArrayNotHasKey('email', $array);
    $this->assertContains(30, $array);  // Value exists
    
    // Count
    $this->assertCount(3, $array);
    $this->assertNotEmpty($array);
    $this->assertEmpty([]);
    
    // Equality
    $this->assertEquals(['a', 'b'], ['a', 'b']);
    $this->assertEqualsCanonicalizing(['b', 'a'], ['a', 'b']);  // Order ignored
}
```

## String Assertions

```php
<?php
public function testStringAssertions(): void
{
    $string = 'Hello, World!';
    
    $this->assertStringStartsWith('Hello', $string);
    $this->assertStringEndsWith('!', $string);
    $this->assertStringContainsString('World', $string);
    $this->assertStringNotContainsString('Goodbye', $string);
    
    // Case insensitive
    $this->assertStringContainsStringIgnoringCase('world', $string);
    
    // Regex
    $this->assertMatchesRegularExpression('/Hello.*World/', $string);
}
```

## Exception Assertions

```php
<?php
public function testExceptionIsThrown(): void
{
    $this->expectException(InvalidArgumentException::class);
    $this->expectExceptionMessage('Value must be positive');
    $this->expectExceptionCode(400);
    
    // Code that throws
    throw new InvalidArgumentException('Value must be positive', 400);
}

// Alternative: Callback approach
public function testExceptionWithCallback(): void
{
    $exception = null;
    
    try {
        $this->calculator->divide(1, 0);
    } catch (DivisionByZeroError $e) {
        $exception = $e;
    }
    
    $this->assertNotNull($exception);
    $this->assertStringContainsString('zero', $exception->getMessage());
}
```

## Custom Failure Messages

```php
<?php
public function testWithCustomMessage(): void
{
    $actual = 2 + 2;
    $expected = 5;
    
    $this->assertEquals(
        $expected,
        $actual,
        'Math is broken: 2 + 2 should equal 5 (just kidding)'
    );
}
```

## Code Examples

**Comprehensive test examples with various assertions**

```php
<?php
declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;
use App\UserValidator;
use App\User;

class UserValidatorTest extends TestCase
{
    private UserValidator $validator;
    
    protected function setUp(): void
    {
        $this->validator = new UserValidator();
    }
    
    public function testValidEmailPasses(): void
    {
        $result = $this->validator->validateEmail('john@example.com');
        $this->assertTrue($result);
    }
    
    public function testInvalidEmailFails(): void
    {
        $result = $this->validator->validateEmail('not-an-email');
        $this->assertFalse($result);
    }
    
    /**
     * @dataProvider invalidEmailProvider
     */
    public function testVariousInvalidEmails(string $email): void
    {
        $this->assertFalse($this->validator->validateEmail($email));
    }
    
    public static function invalidEmailProvider(): array
    {
        return [
            'missing @' => ['invalid'],
            'missing domain' => ['test@'],
            'missing local' => ['@example.com'],
            'spaces' => ['test @example.com'],
            'double @' => ['test@@example.com'],
        ];
    }
    
    public function testPasswordStrength(): void
    {
        // Weak password
        $result = $this->validator->validatePassword('123');
        $this->assertFalse($result['valid']);
        $this->assertArrayHasKey('errors', $result);
        $this->assertContains('Password must be at least 8 characters', $result['errors']);
        
        // Strong password
        $result = $this->validator->validatePassword('SecureP@ss123');
        $this->assertTrue($result['valid']);
        $this->assertEmpty($result['errors']);
    }
    
    public function testValidateUserThrowsOnInvalidData(): void
    {
        $this->expectException(\InvalidArgumentException::class);
        $this->expectExceptionMessage('Email is required');
        
        $this->validator->validateUser(['name' => 'John']);
    }
}
?>
```


---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*