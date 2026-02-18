---
source_course: "php-testing"
source_lesson: "php-testing-data-providers"
---

# Data Providers

Data providers let you run the same test with different inputs, reducing code duplication.

## Basic Data Provider

```php
<?php
class MathTest extends TestCase
{
    /**
     * @dataProvider additionProvider
     */
    public function testAdd(int $a, int $b, int $expected): void
    {
        $calculator = new Calculator();
        $this->assertEquals($expected, $calculator->add($a, $b));
    }
    
    public static function additionProvider(): array
    {
        return [
            'positive numbers' => [1, 2, 3],
            'negative numbers' => [-1, -2, -3],
            'mixed signs' => [-1, 2, 1],
            'with zero' => [0, 5, 5],
        ];
    }
}

// Output:
// ✓ testAdd with positive numbers
// ✓ testAdd with negative numbers
// ✓ testAdd with mixed signs
// ✓ testAdd with with zero
```

## Using PHP 8 Attributes

```php
<?php
use PHPUnit\Framework\Attributes\DataProvider;

class ValidatorTest extends TestCase
{
    #[DataProvider('emailProvider')]
    public function testEmailValidation(string $email, bool $expected): void
    {
        $validator = new EmailValidator();
        $this->assertEquals($expected, $validator->isValid($email));
    }
    
    public static function emailProvider(): array
    {
        return [
            'valid email' => ['user@example.com', true],
            'valid with subdomain' => ['user@mail.example.com', true],
            'valid with plus' => ['user+tag@example.com', true],
            'missing @' => ['userexample.com', false],
            'missing domain' => ['user@', false],
            'invalid chars' => ['user<>@example.com', false],
        ];
    }
}
```

## Generator Data Provider

```php
<?php
class LargeDataTest extends TestCase
{
    /**
     * @dataProvider largeNumberProvider
     */
    public function testSquareRoot(float $input, float $expected): void
    {
        $this->assertEqualsWithDelta($expected, sqrt($input), 0.0001);
    }
    
    public static function largeNumberProvider(): \Generator
    {
        // Use generator for large datasets (memory efficient)
        for ($i = 1; $i <= 100; $i++) {
            yield "sqrt of $i" => [$i * $i, (float) $i];
        }
    }
}
```

## Multiple Data Providers

```php
<?php
class CombinedTest extends TestCase
{
    /**
     * @dataProvider validUsernames
     * @dataProvider validUsernames2
     */
    public function testUsernameValidation(string $username): void
    {
        $this->assertTrue($this->validator->isValidUsername($username));
    }
    
    public static function validUsernames(): array
    {
        return [
            ['john_doe'],
            ['jane123'],
        ];
    }
    
    public static function validUsernames2(): array
    {
        return [
            ['user_2024'],
            ['testUser'],
        ];
    }
}
```

## Code Examples

**Complete test with data providers for slug generation**

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;
use PHPUnit\Framework\Attributes\DataProvider;

class SlugGeneratorTest extends TestCase
{
    private SlugGenerator $generator;
    
    protected function setUp(): void
    {
        $this->generator = new SlugGenerator();
    }
    
    #[DataProvider('slugProvider')]
    public function testGenerateSlug(string $input, string $expected): void
    {
        $this->assertEquals($expected, $this->generator->generate($input));
    }
    
    public static function slugProvider(): array
    {
        return [
            'lowercase' => ['Hello World', 'hello-world'],
            'special chars' => ['Hello! World?', 'hello-world'],
            'multiple spaces' => ['Hello    World', 'hello-world'],
            'unicode' => ['Héllo Wörld', 'hello-world'],
            'numbers' => ['Product 123', 'product-123'],
            'already slug' => ['hello-world', 'hello-world'],
            'leading/trailing spaces' => ['  Hello World  ', 'hello-world'],
            'ampersand' => ['Salt & Pepper', 'salt-and-pepper'],
            'empty' => ['', ''],
        ];
    }
    
    #[DataProvider('maxLengthProvider')]
    public function testSlugMaxLength(string $input, int $maxLength, string $expected): void
    {
        $this->assertEquals(
            $expected,
            $this->generator->generate($input, $maxLength)
        );
    }
    
    public static function maxLengthProvider(): array
    {
        return [
            'truncate long' => ['This is a very long title', 10, 'this-is-a'],
            'no truncate short' => ['Short', 100, 'short'],
            'exact length' => ['Hello', 5, 'hello'],
        ];
    }
}

// SlugGenerator implementation
class SlugGenerator
{
    public function generate(string $text, int $maxLength = 200): string
    {
        // Convert to lowercase
        $slug = strtolower($text);
        
        // Replace & with 'and'
        $slug = str_replace('&', 'and', $slug);
        
        // Remove accents
        $slug = iconv('UTF-8', 'ASCII//TRANSLIT//IGNORE', $slug) ?: $slug;
        
        // Replace non-alphanumeric with hyphens
        $slug = preg_replace('/[^a-z0-9]+/', '-', $slug);
        
        // Remove leading/trailing hyphens
        $slug = trim($slug, '-');
        
        // Truncate
        if (strlen($slug) > $maxLength) {
            $slug = substr($slug, 0, $maxLength);
            $slug = rtrim($slug, '-');
        }
        
        return $slug;
    }
}
?>
```


---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*