---
source_course: "php-api-development"
source_lesson: "php-api-development-openapi-docs"
---

# OpenAPI/Swagger Documentation

## Introduction
Great APIs deserve great documentation. The OpenAPI Specification (formerly Swagger) is the industry standard for describing REST APIs in a machine-readable format. Once you write an OpenAPI definition, you can auto-generate interactive docs, client SDKs, and validation middleware.

## Key Concepts
- **OpenAPI Specification (OAS)**: A language-agnostic standard for describing HTTP APIs, currently at version 3.1.
- **Swagger UI**: An interactive documentation tool that renders an OpenAPI spec as a browsable, testable web page.
- **Schema Object**: An OpenAPI component that defines the shape of request bodies, response payloads, and parameters using JSON Schema.
- **Path Object**: An OpenAPI component that describes a single endpoint, including its HTTP method, parameters, request body, and responses.

## Real World Context
Every developer who has struggled with an undocumented API knows the pain. OpenAPI eliminates guesswork by providing a single source of truth that stays in sync with your code. Stripe's API reference, one of the most praised in the industry, is generated from an OpenAPI spec.

## Deep Dive

### OpenAPI Spec Structure

An OpenAPI document has three main sections: metadata, paths (endpoints), and components (reusable schemas). Here is a minimal spec for a products API:

```yaml
openapi: "3.1.0"
info:
  title: Product API
  version: "1.0.0"
  description: API for managing products
servers:
  - url: https://api.example.com/v1
paths:
  /products:
    get:
      summary: List all products
      operationId: listProducts
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: per_page
          in: query
          schema:
            type: integer
            default: 20
      responses:
        "200":
          description: Successful response
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ProductList"
components:
  schemas:
    Product:
      type: object
      required: [id, name, price]
      properties:
        id:
          type: integer
        name:
          type: string
        price:
          type: number
          format: float
    ProductList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: "#/components/schemas/Product"
        meta:
          type: object
          properties:
            total:
              type: integer
            page:
              type: integer
```

The `$ref` keyword lets you reference reusable schemas defined in the `components` section. This avoids repetition when the same data shape appears in multiple endpoints.

### Writing Endpoint Definitions in JSON

OpenAPI specs can also be written in JSON. Here is the same products endpoint in JSON format:

```json
{
  "openapi": "3.1.0",
  "info": {
    "title": "Product API",
    "version": "1.0.0"
  },
  "paths": {
    "/products/{id}": {
      "get": {
        "summary": "Get a product by ID",
        "operationId": "getProduct",
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "schema": { "type": "integer" }
          }
        ],
        "responses": {
          "200": {
            "description": "Product found",
            "content": {
              "application/json": {
                "schema": { "$ref": "#/components/schemas/Product" }
              }
            }
          },
          "404": {
            "description": "Product not found"
          }
        }
      }
    }
  }
}
```

Both YAML and JSON are fully supported. YAML is more readable for hand-written specs, while JSON is easier to generate programmatically from PHP.

### Describing Request Bodies

POST and PUT endpoints need request body definitions. Here is how to describe a product creation endpoint:

```yaml
paths:
  /products:
    post:
      summary: Create a new product
      operationId: createProduct
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [name, price]
              properties:
                name:
                  type: string
                  minLength: 1
                  maxLength: 255
                price:
                  type: number
                  minimum: 0
                category:
                  type: string
                  enum: [electronics, clothing, food]
      responses:
        "201":
          description: Product created
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Product"
        "422":
          description: Validation error
```

The `requestBody` section specifies the expected payload format, including required fields, data types, and validation constraints like `minLength` and `enum`.

### Serving the Spec from PHP

You can serve your OpenAPI spec directly from a PHP endpoint so that tools like Swagger UI can consume it:

```php
<?php
// GET /api/v1/openapi.json
function serveOpenApiSpec(): never
{
    $specPath = __DIR__ . '/../docs/openapi.yaml';
    $spec = yaml_parse_file($specPath);

    header('Content-Type: application/json');
    header('Access-Control-Allow-Origin: *');
    echo json_encode($spec, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES);
    exit;
}
```

This endpoint reads the YAML spec, converts it to JSON, and serves it with CORS headers so Swagger UI hosted on a different domain can fetch it.

> **Note:** `yaml_parse_file()` requires the PECL `yaml` extension (`pecl install yaml`). Alternatively, use `symfony/yaml` via Composer.

## Common Pitfalls
1. **Letting docs drift from code** — A spec that does not match the actual API behavior is worse than no docs at all. Use automated tests that validate responses against your OpenAPI spec to catch drift.
2. **Skipping error response definitions** — Many developers document the happy path (200) but forget to describe 400, 404, and 422 responses. Clients need to know what error shapes to expect.

## Best Practices
1. **Use `$ref` for reusable schemas** — Define data shapes once in `components/schemas` and reference them everywhere. This keeps your spec DRY and ensures consistency across endpoints.
2. **Include example values** — Add `example` fields to schemas so Swagger UI shows realistic sample data instead of generic placeholders.

## Summary
- OpenAPI (formerly Swagger) is the industry standard for describing REST APIs in a machine-readable format.
- Specs can be written in YAML or JSON and describe endpoints, parameters, request bodies, and responses.
- The `$ref` keyword enables reusable schema definitions to avoid repetition.
- Swagger UI renders OpenAPI specs as interactive, testable documentation pages.
- Always document error responses alongside success cases.

## Code Examples

**Serving an OpenAPI YAML spec as JSON from a PHP endpoint for consumption by Swagger UI and other tools**

```php
<?php
// Serve the OpenAPI spec as JSON from a PHP endpoint
function serveOpenApiSpec(): never
{
    $specPath = __DIR__ . '/../docs/openapi.yaml';

    if (!file_exists($specPath)) {
        http_response_code(500);
        echo json_encode(['error' => 'OpenAPI spec not found']);
        exit;
    }

    $spec = yaml_parse_file($specPath);

    header('Content-Type: application/json');
    header('Access-Control-Allow-Origin: *');
    echo json_encode($spec, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES);
    exit;
}

// Register the endpoint
$router->get('/api/v1/openapi.json', 'serveOpenApiSpec');
```


## Resources

- [PHP yaml_parse_file — Parse YAML Files](https://www.php.net/manual/en/function.yaml-parse-file.php) — Official PHP documentation for parsing YAML files used in OpenAPI spec loading

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*