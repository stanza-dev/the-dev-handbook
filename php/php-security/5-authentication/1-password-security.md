---
source_course: "php-security"
source_lesson: "php-security-password-security"
---

# Password Hashing & Storage

Never store passwords in plain text. Use proper cryptographic hashing.

## The password_hash() Function

```php
<?php
// Creating a hash
$password = 'user_password_here';
$hash = password_hash($password, PASSWORD_DEFAULT);
// Output: $2y$10$... (60+ characters)

// PASSWORD_DEFAULT uses bcrypt (currently)
// Automatically generates a secure salt
// Future-proof: algorithm may change
```

## Verifying Passwords

```php
<?php
$submitted = $_POST['password'];
$storedHash = $user['password_hash'];  // From database

if (password_verify($submitted, $storedHash)) {
    // Password is correct
    login($user);
} else {
    // Password is wrong
    // Don't reveal which field was wrong!
    throw new AuthenticationException('Invalid credentials');
}
```

## Algorithm Options

```php
<?php
// Bcrypt (default, recommended for most cases)
$hash = password_hash($password, PASSWORD_BCRYPT, [
    'cost' => 12  // Higher = slower = more secure
]);

// Argon2id (PHP 7.3+, best for new projects)
$hash = password_hash($password, PASSWORD_ARGON2ID, [
    'memory_cost' => 65536,  // 64 MB
    'time_cost' => 4,        // 4 iterations
    'threads' => 3           // 3 threads
]);
```

## Rehashing When Needed

```php
<?php
function login(string $password, string $hash): bool {
    if (!password_verify($password, $hash)) {
        return false;
    }
    
    // Check if hash needs upgrade
    if (password_needs_rehash($hash, PASSWORD_DEFAULT)) {
        $newHash = password_hash($password, PASSWORD_DEFAULT);
        updateUserPasswordHash($user->id, $newHash);
    }
    
    return true;
}
```

## Common Mistakes

```php
<?php
// WRONG: Plain text storage
$user->password = $password;

// WRONG: Simple hashing (crackable)
$hash = md5($password);
$hash = sha1($password);
$hash = hash('sha256', $password);

// WRONG: Unsalted hash (rainbow tables)
$hash = hash('sha256', $password);

// WRONG: Static salt (if leaked, all passwords vulnerable)
$hash = hash('sha256', 'static_salt' . $password);

// RIGHT: password_hash() with unique salt per password
$hash = password_hash($password, PASSWORD_DEFAULT);
```

## Timing Attacks

```php
<?php
// WRONG: Early return reveals information
if (strlen($password) < 8) {
    return false;  // Fast response = password too short
}

// WRONG: String comparison timing
if ($hash === $expected) {  // Timing varies by position
    return true;
}

// RIGHT: Constant-time comparison
if (hash_equals($expected, $hash)) {
    return true;
}
```

## Code Examples

**Complete password service with strength validation**

```php
<?php
declare(strict_types=1);

class PasswordService {
    private const ALGORITHM = PASSWORD_ARGON2ID;
    private const OPTIONS = [
        'memory_cost' => 65536,
        'time_cost' => 4,
        'threads' => 3,
    ];
    
    public function hash(string $password): string {
        $this->validateStrength($password);
        return password_hash($password, self::ALGORITHM, self::OPTIONS);
    }
    
    public function verify(string $password, string $hash): bool {
        return password_verify($password, $hash);
    }
    
    public function needsRehash(string $hash): bool {
        return password_needs_rehash($hash, self::ALGORITHM, self::OPTIONS);
    }
    
    public function validateStrength(string $password): void {
        $errors = [];
        
        if (strlen($password) < 8) {
            $errors[] = 'Password must be at least 8 characters';
        }
        
        if (!preg_match('/[A-Z]/', $password)) {
            $errors[] = 'Password must contain an uppercase letter';
        }
        
        if (!preg_match('/[a-z]/', $password)) {
            $errors[] = 'Password must contain a lowercase letter';
        }
        
        if (!preg_match('/[0-9]/', $password)) {
            $errors[] = 'Password must contain a number';
        }
        
        if ($errors) {
            throw new WeakPasswordException(implode('. ', $errors));
        }
    }
}

// Registration
$passwordService = new PasswordService();
$hash = $passwordService->hash($_POST['password']);
$stmt->execute(['password_hash' => $hash]);

// Login
if ($passwordService->verify($_POST['password'], $user['password_hash'])) {
    if ($passwordService->needsRehash($user['password_hash'])) {
        $newHash = $passwordService->hash($_POST['password']);
        // Update in database
    }
    // Login successful
}
?>
```


## Resources

- [Password Hashing](https://www.php.net/manual/en/function.password-hash.php) — PHP password hashing documentation

---

> 📘 *This lesson is part of the [PHP Security Engineering](https://stanza.dev/courses/php-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*