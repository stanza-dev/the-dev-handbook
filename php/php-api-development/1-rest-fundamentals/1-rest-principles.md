---
source_course: "php-api-development"
source_lesson: "php-api-development-rest-principles"
---

# REST Architecture Principles

REST (Representational State Transfer) is an architectural style for building web APIs that are scalable, maintainable, and intuitive.

## Core Principles

### 1. Client-Server Separation
Clients and servers are independent. The client handles UI/UX, server handles data/logic.

### 2. Statelessness
Each request contains all information needed. Server doesn't store client state between requests.

```php
<?php
// BAD: Server stores state
$_SESSION['last_action'] = 'viewed_products';
// Next request depends on this

// GOOD: Request is self-contained
// GET /products?page=2&sort=price&filter=active
// All context in the request itself
```

### 3. Resource-Based URLs

```
# Resources (nouns, not verbs)
GET    /users          # List users
GET    /users/123      # Get user 123
POST   /users          # Create user
PUT    /users/123      # Replace user 123
PATCH  /users/123      # Update user 123 partially
DELETE /users/123      # Delete user 123

# Nested resources
GET    /users/123/orders        # User's orders
GET    /users/123/orders/456    # Specific order
```

### 4. HTTP Methods (Verbs)

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Read | Yes | Yes |
| POST | Create | No | No |
| PUT | Replace | Yes | No |
| PATCH | Partial update | Yes | No |
| DELETE | Remove | Yes | No |

### 5. HTTP Status Codes

```php
<?php
// Success
http_response_code(200);  // OK
http_response_code(201);  // Created
http_response_code(204);  // No Content (DELETE)

// Client Errors
http_response_code(400);  // Bad Request
http_response_code(401);  // Unauthorized
http_response_code(403);  // Forbidden
http_response_code(404);  // Not Found
http_response_code(422);  // Unprocessable Entity (validation)

// Server Errors
http_response_code(500);  // Internal Server Error
http_response_code(503);  // Service Unavailable
```

## URL Design Best Practices

```
# Good URLs
GET  /products
GET  /products/123
GET  /products?category=electronics
GET  /products?sort=price&order=desc
GET  /users/123/orders

# Bad URLs
GET  /getProducts         # Verb in URL
GET  /product/123         # Inconsistent plural
POST /users/create        # Action in URL
GET  /users/123/getOrders # Verb in URL
```

## Resources

- [HTTP Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) — HTTP methods reference

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*