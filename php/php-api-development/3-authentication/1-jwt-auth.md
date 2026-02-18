---
source_course: "php-api-development"
source_lesson: "php-api-development-jwt-auth"
---

# JWT Authentication

JSON Web Tokens (JWT) are a stateless authentication method ideal for APIs.

## JWT Structure

```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.  <- Header (base64)
eyJ1c2VyX2lkIjoxLCJleHAiOjE3MDk5MjM0NTZ9.  <- Payload (base64)
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  <- Signature
```

## Creating JWTs Without Libraries

```php
<?php
class JWT {
    private string $secret;
    
    public function __construct(string $secret) {
        $this->secret = $secret;
    }
    
    public function encode(array $payload, int $expiresIn = 3600): string {
        $header = ['alg' => 'HS256', 'typ' => 'JWT'];
        
        $payload['iat'] = time();
        $payload['exp'] = time() + $expiresIn;
        
        $headerEncoded = $this->base64UrlEncode(json_encode($header));
        $payloadEncoded = $this->base64UrlEncode(json_encode($payload));
        
        $signature = hash_hmac(
            'sha256',
            "$headerEncoded.$payloadEncoded",
            $this->secret,
            true
        );
        $signatureEncoded = $this->base64UrlEncode($signature);
        
        return "$headerEncoded.$payloadEncoded.$signatureEncoded";
    }
    
    public function decode(string $token): ?array {
        $parts = explode('.', $token);
        
        if (count($parts) !== 3) {
            return null;
        }
        
        [$headerEncoded, $payloadEncoded, $signatureEncoded] = $parts;
        
        // Verify signature
        $expectedSignature = hash_hmac(
            'sha256',
            "$headerEncoded.$payloadEncoded",
            $this->secret,
            true
        );
        
        if (!hash_equals($this->base64UrlEncode($expectedSignature), $signatureEncoded)) {
            return null;  // Invalid signature
        }
        
        $payload = json_decode($this->base64UrlDecode($payloadEncoded), true);
        
        // Check expiration
        if (isset($payload['exp']) && $payload['exp'] < time()) {
            return null;  // Token expired
        }
        
        return $payload;
    }
    
    private function base64UrlEncode(string $data): string {
        return rtrim(strtr(base64_encode($data), '+/', '-_'), '=');
    }
    
    private function base64UrlDecode(string $data): string {
        return base64_decode(strtr($data, '-_', '+/'));
    }
}
```

## Authentication Flow

```php
<?php
// Login endpoint
$router->post('/auth/login', function() use ($jwt, $userRepo) {
    $input = json_decode(file_get_contents('php://input'), true);
    
    $user = $userRepo->findByEmail($input['email'] ?? '');
    
    if (!$user || !password_verify($input['password'] ?? '', $user->passwordHash)) {
        http_response_code(401);
        return ['error' => 'Invalid credentials'];
    }
    
    $token = $jwt->encode([
        'user_id' => $user->id,
        'email' => $user->email,
        'role' => $user->role,
    ], 86400);  // 24 hours
    
    return [
        'token' => $token,
        'expires_in' => 86400,
        'token_type' => 'Bearer',
    ];
});
```

## Reading the Token

```php
<?php
function getAuthUser(JWT $jwt): ?array {
    $header = $_SERVER['HTTP_AUTHORIZATION'] ?? '';
    
    if (!preg_match('/^Bearer\s+(\S+)$/', $header, $matches)) {
        return null;
    }
    
    return $jwt->decode($matches[1]);
}

// Protected endpoint
$router->get('/me', function() use ($jwt) {
    $user = getAuthUser($jwt);
    
    if (!$user) {
        http_response_code(401);
        return ['error' => 'Unauthorized'];
    }
    
    return ['user' => $user];
});
```

## Resources

- [JWT Introduction](https://jwt.io/introduction) — JWT standard introduction

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*