---
source_course: "django-rest-api"
source_lesson: "django-rest-api-rest-principles"
---

# REST API Principles

## Introduction

Every time you check the weather on your phone, scroll through social media, or make an online purchase, you're interacting with REST APIs. REST (Representational State Transfer) has become the lingua franca of modern web development, enabling seamless communication between applications worldwide.

## Key Concepts

**REST (Representational State Transfer)** is an architectural style that defines a set of constraints for creating web services. A **RESTful API** is an API that adheres to these constraints, using HTTP protocols to perform operations on resources.

**Resource**: Any piece of information that can be named—users, articles, products, orders. Each resource is identified by a unique URL.

**Endpoint**: A specific URL where a resource can be accessed.

**HTTP Methods**: The verbs that define what operation to perform (GET, POST, PUT, PATCH, DELETE).

## Real World Context

In production applications, REST APIs are the backbone of:
- **Mobile apps** communicating with backend servers
- **Single Page Applications (SPAs)** fetching and updating data
- **Microservices** talking to each other
- **Third-party integrations** (payment gateways, shipping APIs, social login)

Without understanding REST principles, you'll build APIs that are inconsistent, hard to maintain, and frustrating for other developers to use.

## Deep Dive

### The Six REST Constraints

**1. Client-Server Separation**
```
Client (React, Mobile App)  <-->  Server (Django API)
     UI concerns                   Data & Logic
```

**2. Statelessness** - Each request contains all information needed:
```
# BAD: Server remembers state
GET /next-page  (server must remember current page)

# GOOD: Request contains all info
GET /articles?page=2&per_page=10
```

**3. Uniform Interface** - Consistent patterns across endpoints:

| Resource | GET | POST | PUT/PATCH | DELETE |
|----------|-----|------|-----------|--------|
| `/articles` | List all | Create new | - | - |
| `/articles/1` | Get one | - | Update | Delete |

**4. Resource-Based URLs** - Nouns, not verbs:
```
# GOOD: Resource-based
GET    /articles          # List articles
POST   /articles          # Create article
GET    /articles/1        # Get article 1
DELETE /articles/1        # Delete article 1

# BAD: Action-based
GET /getArticles
POST /deleteArticle?id=1
```

**5. Layered System** - Client doesn't know if it's connected directly to the server or through intermediaries (load balancers, caches).

**6. Cacheable** - Responses must define themselves as cacheable or non-cacheable.

### HTTP Methods and Their Semantics

| Method | Purpose | Idempotent | Safe | Request Body |
|--------|---------|------------|------|-------------|
| GET | Retrieve | Yes | Yes | No |
| POST | Create | No | No | Yes |
| PUT | Replace | Yes | No | Yes |
| PATCH | Update | Yes | No | Yes |
| DELETE | Remove | Yes | No | No |

### HTTP Status Codes

**Success (2xx)**
- `200 OK` - Request succeeded
- `201 Created` - Resource created (return new resource)
- `204 No Content` - Success, no body (typical for DELETE)

**Client Errors (4xx)**
- `400 Bad Request` - Malformed request
- `401 Unauthorized` - Authentication required
- `403 Forbidden` - Authenticated but not authorized
- `404 Not Found` - Resource doesn't exist
- `422 Unprocessable Entity` - Validation failed

**Server Errors (5xx)**
- `500 Internal Server Error` - Generic server error
- `503 Service Unavailable` - Server overloaded or in maintenance

## Common Pitfalls

1. **Using verbs in URLs**: `/api/getUsers` instead of `GET /api/users`. URLs should represent resources (nouns), with HTTP methods providing the action.

2. **Ignoring status codes**: Returning `200 OK` for errors with an error message in the body. Clients rely on status codes for error handling.

3. **Not being stateless**: Storing session state on the server between requests. Each request should be self-contained with all necessary authentication/context.

## Best Practices

1. **Use plural nouns for resources**: `/articles` not `/article`
2. **Use kebab-case for multi-word resources**: `/user-profiles` not `/userProfiles`
3. **Nest resources logically**: `/articles/1/comments` for comments on article 1
4. **Version your API from day one**: `/api/v1/articles`
5. **Return appropriate status codes**: Be specific—use `201` for creation, `204` for deletion
6. **Use JSON consistently**: Set `Content-Type: application/json` headers

## Summary

REST APIs follow six architectural constraints that make them scalable, maintainable, and predictable. The key principles are: use resources (nouns) in URLs, leverage HTTP methods for actions, return appropriate status codes, and keep requests stateless. Mastering these fundamentals will help you design APIs that other developers love to use.

## Resources

- [Django Request and Response Objects](https://docs.djangoproject.com/en/6.0/ref/request-response/) — Official reference for HTTP handling in Django
- [Django URL Dispatcher](https://docs.djangoproject.com/en/6.0/topics/http/urls/) — How Django routes URLs to views

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*