---
source_course: "php-api-development"
source_lesson: "php-api-development-openapi-docs"
---

# OpenAPI Documentation

OpenAPI (formerly Swagger) is the standard for documenting REST APIs.

## OpenAPI Specification Basics

```yaml
openapi: 3.0.3
info:
  title: My API
  version: 1.0.0
  description: A sample API

servers:
  - url: https://api.example.com/v1

paths:
  /users:
    get:
      summary: List users
      operationId: listUsers
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
          format: email
```

## Generating Docs from PHP

```php
<?php
use OpenApi\Attributes as OA;

#[OA\Info(title: 'My API', version: '1.0.0')]
#[OA\Server(url: 'https://api.example.com/v1')]
class OpenApiSpec {}

#[OA\Schema(schema: 'User')]
class User {
    #[OA\Property(type: 'integer')]
    public int $id;
    
    #[OA\Property(type: 'string', maxLength: 255)]
    public string $name;
    
    #[OA\Property(type: 'string', format: 'email')]
    public string $email;
}

class UserController {
    #[OA\Get(
        path: '/users',
        summary: 'List all users',
        tags: ['Users'],
        parameters: [
            new OA\Parameter(
                name: 'page',
                in: 'query',
                schema: new OA\Schema(type: 'integer', default: 1)
            )
        ],
        responses: [
            new OA\Response(
                response: 200,
                description: 'List of users',
                content: new OA\JsonContent(
                    type: 'object',
                    properties: [
                        new OA\Property(
                            property: 'data',
                            type: 'array',
                            items: new OA\Items(ref: '#/components/schemas/User')
                        )
                    ]
                )
            ),
            new OA\Response(response: 401, description: 'Unauthorized')
        ]
    )]
    public function index(): never { /* ... */ }
    
    #[OA\Post(
        path: '/users',
        summary: 'Create a new user',
        tags: ['Users'],
        requestBody: new OA\RequestBody(
            required: true,
            content: new OA\JsonContent(
                required: ['name', 'email', 'password'],
                properties: [
                    new OA\Property(property: 'name', type: 'string'),
                    new OA\Property(property: 'email', type: 'string', format: 'email'),
                    new OA\Property(property: 'password', type: 'string', minLength: 8),
                ]
            )
        ),
        responses: [
            new OA\Response(response: 201, description: 'User created'),
            new OA\Response(response: 422, description: 'Validation error')
        ]
    )]
    public function store(): never { /* ... */ }
}
```

## Serving Documentation

```php
<?php
// Generate OpenAPI JSON
$router->get('/openapi.json', function() {
    $openapi = \OpenApi\Generator::scan([__DIR__ . '/src']);
    
    header('Content-Type: application/json');
    echo $openapi->toJson();
    exit;
});

// Serve Swagger UI
$router->get('/docs', function() {
    ?>
    <!DOCTYPE html>
    <html>
    <head>
        <title>API Documentation</title>
        <link rel="stylesheet" type="text/css" href="https://unpkg.com/swagger-ui-dist/swagger-ui.css">
    </head>
    <body>
        <div id="swagger-ui"></div>
        <script src="https://unpkg.com/swagger-ui-dist/swagger-ui-bundle.js"></script>
        <script>
            SwaggerUIBundle({
                url: '/openapi.json',
                dom_id: '#swagger-ui'
            });
        </script>
    </body>
    </html>
    <?php
    exit;
});
```

## Code Examples

**Manual OpenAPI specification generator**

```php
<?php
declare(strict_types=1);

// Manual OpenAPI spec generator
class OpenApiGenerator {
    private array $spec = [
        'openapi' => '3.0.3',
        'info' => [],
        'servers' => [],
        'paths' => [],
        'components' => ['schemas' => [], 'securitySchemes' => []],
    ];
    
    public function info(string $title, string $version, string $description = ''): self {
        $this->spec['info'] = compact('title', 'version', 'description');
        return $this;
    }
    
    public function server(string $url, string $description = ''): self {
        $this->spec['servers'][] = compact('url', 'description');
        return $this;
    }
    
    public function bearerAuth(): self {
        $this->spec['components']['securitySchemes']['bearerAuth'] = [
            'type' => 'http',
            'scheme' => 'bearer',
            'bearerFormat' => 'JWT',
        ];
        return $this;
    }
    
    public function schema(string $name, array $properties, array $required = []): self {
        $this->spec['components']['schemas'][$name] = [
            'type' => 'object',
            'properties' => $properties,
            'required' => $required,
        ];
        return $this;
    }
    
    public function path(string $path, string $method, array $config): self {
        $this->spec['paths'][$path][strtolower($method)] = $config;
        return $this;
    }
    
    public function toJson(): string {
        return json_encode($this->spec, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES);
    }
    
    public function toYaml(): string {
        return yaml_emit($this->spec);
    }
}

// Build spec
$api = new OpenApiGenerator();

$api->info('My API', '1.0.0', 'A comprehensive REST API')
    ->server('https://api.example.com/v1', 'Production')
    ->server('http://localhost:8080/v1', 'Development')
    ->bearerAuth()
    ->schema('User', [
        'id' => ['type' => 'integer', 'readOnly' => true],
        'name' => ['type' => 'string', 'maxLength' => 255],
        'email' => ['type' => 'string', 'format' => 'email'],
        'created_at' => ['type' => 'string', 'format' => 'date-time', 'readOnly' => true],
    ], ['name', 'email'])
    ->schema('Error', [
        'error' => ['type' => 'string'],
        'details' => ['type' => 'object', 'additionalProperties' => true],
    ])
    ->path('/users', 'GET', [
        'summary' => 'List users',
        'tags' => ['Users'],
        'security' => [['bearerAuth' => []]],
        'parameters' => [
            ['name' => 'page', 'in' => 'query', 'schema' => ['type' => 'integer', 'default' => 1]],
            ['name' => 'limit', 'in' => 'query', 'schema' => ['type' => 'integer', 'default' => 20]],
        ],
        'responses' => [
            '200' => [
                'description' => 'Success',
                'content' => [
                    'application/json' => [
                        'schema' => [
                            'type' => 'object',
                            'properties' => [
                                'data' => ['type' => 'array', 'items' => ['$ref' => '#/components/schemas/User']],
                                'meta' => ['type' => 'object', 'properties' => [
                                    'total' => ['type' => 'integer'],
                                    'page' => ['type' => 'integer'],
                                ]],
                            ],
                        ],
                    ],
                ],
            ],
            '401' => ['description' => 'Unauthorized'],
        ],
    ])
    ->path('/users', 'POST', [
        'summary' => 'Create user',
        'tags' => ['Users'],
        'security' => [['bearerAuth' => []]],
        'requestBody' => [
            'required' => true,
            'content' => [
                'application/json' => [
                    'schema' => ['$ref' => '#/components/schemas/User'],
                ],
            ],
        ],
        'responses' => [
            '201' => ['description' => 'Created'],
            '422' => ['description' => 'Validation error'],
        ],
    ]);

echo $api->toJson();
?>
```


## Resources

- [OpenAPI Specification](https://swagger.io/specification/) — OpenAPI 3.0 specification reference

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*