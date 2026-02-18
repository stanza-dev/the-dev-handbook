---
source_course: "php-testing"
source_lesson: "php-testing-tdd-example"
---

# TDD in Practice

Let's build a password validator using TDD.

## Requirements

- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character

## Step 1: First Test (RED)

```php
<?php
class PasswordValidatorTest extends TestCase
{
    private PasswordValidator $validator;
    
    protected function setUp(): void
    {
        $this->validator = new PasswordValidator();
    }
    
    public function testRejectsShortPassword(): void
    {
        $result = $this->validator->validate('Short1!');
        
        $this->assertFalse($result->isValid);
        $this->assertContains(
            'Password must be at least 8 characters',
            $result->errors
        );
    }
}
```

## Step 2: Minimum Implementation (GREEN)

```php
<?php
class ValidationResult
{
    public function __construct(
        public bool $isValid,
        public array $errors = []
    ) {}
}

class PasswordValidator
{
    public function validate(string $password): ValidationResult
    {
        $errors = [];
        
        if (strlen($password) < 8) {
            $errors[] = 'Password must be at least 8 characters';
        }
        
        return new ValidationResult(
            isValid: empty($errors),
            errors: $errors
        );
    }
}
```

## Step 3: Add More Tests

```php
<?php
public function testRejectsPasswordWithoutUppercase(): void
{
    $result = $this->validator->validate('lowercase1!');
    
    $this->assertFalse($result->isValid);
    $this->assertContains(
        'Password must contain an uppercase letter',
        $result->errors
    );
}

public function testRejectsPasswordWithoutLowercase(): void
{
    $result = $this->validator->validate('UPPERCASE1!');
    
    $this->assertFalse($result->isValid);
}

public function testRejectsPasswordWithoutNumber(): void
{
    $result = $this->validator->validate('NoNumber!');
    
    $this->assertFalse($result->isValid);
}

public function testRejectsPasswordWithoutSpecialChar(): void
{
    $result = $this->validator->validate('NoSpecial1');
    
    $this->assertFalse($result->isValid);
}

public function testAcceptsValidPassword(): void
{
    $result = $this->validator->validate('ValidP@ss1');
    
    $this->assertTrue($result->isValid);
    $this->assertEmpty($result->errors);
}
```

## Step 4: Complete Implementation

```php
<?php
class PasswordValidator
{
    private array $rules = [];
    
    public function __construct()
    {
        $this->rules = [
            ['/^.{8,}$/', 'Password must be at least 8 characters'],
            ['/[A-Z]/', 'Password must contain an uppercase letter'],
            ['/[a-z]/', 'Password must contain a lowercase letter'],
            ['/[0-9]/', 'Password must contain a number'],
            ['/[!@#$%^&*(),.?":{}|<>]/', 'Password must contain a special character'],
        ];
    }
    
    public function validate(string $password): ValidationResult
    {
        $errors = [];
        
        foreach ($this->rules as [$pattern, $message]) {
            if (!preg_match($pattern, $password)) {
                $errors[] = $message;
            }
        }
        
        return new ValidationResult(
            isValid: empty($errors),
            errors: $errors
        );
    }
}
```

## Code Examples

**Complete TDD example: URL shortener**

```php
<?php
declare(strict_types=1);

// TDD Example: Building a URL shortener

// Final test suite
class UrlShortenerTest extends TestCase
{
    private UrlShortener $shortener;
    private InMemoryUrlRepository $repository;
    
    protected function setUp(): void
    {
        $this->repository = new InMemoryUrlRepository();
        $this->shortener = new UrlShortener($this->repository);
    }
    
    public function testShortenUrlReturnsShortCode(): void
    {
        $code = $this->shortener->shorten('https://example.com/very/long/url');
        
        $this->assertNotEmpty($code);
        $this->assertEquals(6, strlen($code));
        $this->assertMatchesRegularExpression('/^[a-zA-Z0-9]+$/', $code);
    }
    
    public function testExpandReturnsOriginalUrl(): void
    {
        $url = 'https://example.com/test';
        $code = $this->shortener->shorten($url);
        
        $expanded = $this->shortener->expand($code);
        
        $this->assertEquals($url, $expanded);
    }
    
    public function testSameUrlReturnsSameCode(): void
    {
        $url = 'https://example.com/test';
        
        $code1 = $this->shortener->shorten($url);
        $code2 = $this->shortener->shorten($url);
        
        $this->assertEquals($code1, $code2);
    }
    
    public function testExpandUnknownCodeThrows(): void
    {
        $this->expectException(NotFoundException::class);
        
        $this->shortener->expand('notfound');
    }
    
    public function testRejectsInvalidUrl(): void
    {
        $this->expectException(InvalidArgumentException::class);
        
        $this->shortener->shorten('not-a-valid-url');
    }
}

// Implementation built through TDD
class UrlShortener
{
    public function __construct(
        private UrlRepository $repository,
        private int $codeLength = 6
    ) {}
    
    public function shorten(string $url): string
    {
        if (!filter_var($url, FILTER_VALIDATE_URL)) {
            throw new InvalidArgumentException('Invalid URL');
        }
        
        // Check if already shortened
        $existing = $this->repository->findByUrl($url);
        if ($existing) {
            return $existing->code;
        }
        
        // Generate new code
        do {
            $code = $this->generateCode();
        } while ($this->repository->findByCode($code));
        
        $this->repository->save(new ShortUrl($code, $url));
        
        return $code;
    }
    
    public function expand(string $code): string
    {
        $shortUrl = $this->repository->findByCode($code);
        
        if (!$shortUrl) {
            throw new NotFoundException("URL not found for code: $code");
        }
        
        return $shortUrl->url;
    }
    
    private function generateCode(): string
    {
        $chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
        $code = '';
        for ($i = 0; $i < $this->codeLength; $i++) {
            $code .= $chars[random_int(0, strlen($chars) - 1)];
        }
        return $code;
    }
}
?>
```


---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*