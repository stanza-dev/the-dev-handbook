---
source_course: "php-api-development"
source_lesson: "php-api-development-auto-docs"
---

# Auto-generating API Documentation

## Introduction
Writing OpenAPI specs by hand works for small APIs, but it does not scale. As your API grows to dozens of endpoints, manually maintaining a YAML file becomes error-prone. Auto-generating documentation from your PHP source code keeps your docs accurate and your developers happy.

## Key Concepts
- **PHP Attributes**: Native metadata annotations (PHP 8.0+) attached to classes, methods, and parameters that tools can read at runtime.
- **Doc Generation**: The process of extracting route definitions, type information, and annotations from source code to produce an OpenAPI spec automatically.
- **Swagger UI**: A web-based tool that renders an OpenAPI spec as interactive documentation where developers can try out API calls directly.
- **Spec Validation**: Checking that a generated OpenAPI spec conforms to the OpenAPI standard and matches actual API behavior.

## Real World Context
Teams that auto-generate docs from code eliminate an entire class of bugs: documentation drift. When a developer adds a new parameter or changes a response field, the docs update automatically. Laravel, Symfony, and API Platform all support attribute-driven doc generation.

## Deep Dive

### PHP Attributes for API Documentation

PHP 8.0 introduced native attributes that replace docblock annotations. You can define custom attributes to annotate your API endpoints with metadata used for documentation generation.

Here are custom attribute classes for describing API endpoints:

```php
<?php
#[Attribute(Attribute::TARGET_METHOD)]
class ApiEndpoint
{
    public function __construct(
        public readonly string $summary,
        public readonly string $description = '',
        public readonly string $method = 'GET',
        public readonly string $path = '',
        public readonly array $tags = [],
    ) {}
}

#[Attribute(Attribute::TARGET_METHOD | Attribute::IS_REPEATABLE)]
class ApiResponse
{
    public function __construct(
        public readonly int $statusCode,
        public readonly string $description,
        public readonly ?string $schemaRef = null,
    ) {}
}

#[Attribute(Attribute::TARGET_PARAMETER)]
class ApiParam
{
    public function __construct(
        public readonly string $description,
        public readonly string $in = 'query',
        public readonly bool $required = false,
    ) {}
}
```

These attributes define the metadata that a doc generator will extract. The `Attribute::IS_REPEATABLE` flag on `ApiResponse` allows multiple response annotations on a single method.

### Annotating Controllers

With the attributes defined, you annotate your controller methods to describe each endpoint:

```php
<?php
class ProductController extends Controller
{
    #[ApiEndpoint(
        summary: 'List all products',
        method: 'GET',
        path: '/api/v1/products',
        tags: ['Products'],
    )]
    #[ApiResponse(200, 'Product list', '#/components/schemas/ProductList')]
    public function index(
        #[ApiParam('Page number', in: 'query')] int $page = 1,
        #[ApiParam('Items per page', in: 'query')] int $perPage = 20,
    ): never {
        $products = $this->products->paginate($page, $perPage);
        $this->json($products);
    }

    #[ApiEndpoint(
        summary: 'Get a single product',
        method: 'GET',
        path: '/api/v1/products/{id}',
        tags: ['Products'],
    )]
    #[ApiResponse(200, 'Product found', '#/components/schemas/Product')]
    #[ApiResponse(404, 'Product not found')]
    public function show(
        #[ApiParam('Product ID', in: 'path', required: true)] int $id,
    ): never {
        $product = $this->products->find($id)
            ?? throw new NotFoundException('Product not found');
        $this->json(['data' => $product]);
    }
}
```

Each method is fully described with its HTTP method, path, parameters, and possible responses. A doc generator reads these attributes via reflection.

### Building a Spec Generator

The doc generator uses PHP Reflection to scan controllers and extract attribute metadata into an OpenAPI-compatible array:

```php
<?php
class OpenApiGenerator
{
    private array $spec = [
        'openapi' => '3.1.0',
        'info' => ['title' => '', 'version' => '1.0.0'],
        'paths' => [],
    ];

    public function __construct(string $title, string $version)
    {
        $this->spec['info']['title'] = $title;
        $this->spec['info']['version'] = $version;
    }

    public function scanController(string $className): void
    {
        $reflection = new ReflectionClass($className);

        foreach ($reflection->getMethods() as $method) {
            $endpointAttrs = $method->getAttributes(ApiEndpoint::class);
            if (empty($endpointAttrs)) {
                continue;
            }

            $endpoint = $endpointAttrs[0]->newInstance();
            $httpMethod = strtolower($endpoint->method);

            $operation = [
                'summary' => $endpoint->summary,
                'tags' => $endpoint->tags,
                'parameters' => [],
                'responses' => [],
            ];

            // Extract parameter attributes
            foreach ($method->getParameters() as $param) {
                $paramAttrs = $param->getAttributes(ApiParam::class);
                if (!empty($paramAttrs)) {
                    $apiParam = $paramAttrs[0]->newInstance();
                    $operation['parameters'][] = [
                        'name' => $param->getName(),
                        'in' => $apiParam->in,
                        'required' => $apiParam->required,
                        'description' => $apiParam->description,
                        'schema' => ['type' => $this->phpTypeToSchema($param)],
                    ];
                }
            }

            // Extract response attributes
            foreach ($method->getAttributes(ApiResponse::class) as $attr) {
                $response = $attr->newInstance();
                $operation['responses'][(string) $response->statusCode] = [
                    'description' => $response->description,
                ];
                if ($response->schemaRef) {
                    $operation['responses'][(string) $response->statusCode]['content'] = [
                        'application/json' => [
                            'schema' => ['\$ref' => $response->schemaRef],
                        ],
                    ];
                }
            }

            $this->spec['paths'][$endpoint->path][$httpMethod] = $operation;
        }
    }

    private function phpTypeToSchema(ReflectionParameter $param): string
    {
        $type = $param->getType()?->getName() ?? 'string';
        return match ($type) {
            'int' => 'integer',
            'float' => 'number',
            'bool' => 'boolean',
            default => 'string',
        };
    }

    public function toJson(): string
    {
        return json_encode($this->spec, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES);
    }
}
```

The generator scans each controller method for `ApiEndpoint`, `ApiParam`, and `ApiResponse` attributes. It maps PHP types to OpenAPI schema types and builds a complete spec array.

### Integrating Swagger UI

Once you have a generated spec, serve Swagger UI to render it as interactive documentation:

```php
<?php
// Serve Swagger UI at /docs
function serveSwaggerUI(): never
{
    $specUrl = '/api/v1/openapi.json';

    $html = <<<HTML
    <!DOCTYPE html>
    <html>
    <head>
        <title>API Documentation</title>
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/swagger-ui-dist/swagger-ui.css">
    </head>
    <body>
        <div id="swagger-ui"></div>
        <script src="https://cdn.jsdelivr.net/npm/swagger-ui-dist/swagger-ui-bundle.js"></script>
        <script>
            SwaggerUIBundle({
                url: '{$specUrl}',
                dom_id: '#swagger-ui',
                presets: [SwaggerUIBundle.presets.apis],
            });
        </script>
    </body>
    </html>
    HTML;

    header('Content-Type: text/html');
    echo $html;
    exit;
}
```

This creates a self-contained documentation page. Point the `url` to your generated spec endpoint and Swagger UI renders every endpoint with try-it-out functionality.

## Common Pitfalls
1. **Not validating the generated spec** — A malformed OpenAPI spec will cause Swagger UI to fail silently or show incorrect information. Run your generated spec through a validator like `swagger-cli validate` to catch structural issues.
2. **Forgetting to regenerate after changes** — If you build the spec at deploy time, make sure your CI/CD pipeline includes the generation step. Stale docs undermine trust in your API.

## Best Practices
1. **Generate on every build** — Include spec generation in your build pipeline so documentation is always current. This eliminates manual steps and documentation drift.
2. **Version your spec alongside your code** — Commit the generated spec to your repository so code reviewers can see documentation changes in pull requests.

## Summary
- PHP 8.0+ attributes let you annotate controllers with API metadata directly in source code.
- A spec generator uses Reflection to extract attributes and build an OpenAPI-compliant document.
- Swagger UI renders the generated spec as interactive documentation with try-it-out functionality.
- Automate spec generation in your CI/CD pipeline to prevent documentation from drifting out of sync.
- Always validate generated specs to catch structural errors before publishing.

## Code Examples

**Annotated controller with custom PHP attributes and automated spec generation — the generator scans controllers and outputs a valid OpenAPI JSON file**

```php
<?php
// Annotating a controller method with custom API attributes
class OrderController extends Controller
{
    #[ApiEndpoint(
        summary: 'Create a new order',
        method: 'POST',
        path: '/api/v1/orders',
        tags: ['Orders'],
    )]
    #[ApiResponse(201, 'Order created', '#/components/schemas/Order')]
    #[ApiResponse(422, 'Validation failed')]
    public function store(
        #[ApiParam('Customer ID', in: 'query', required: true)] int $customerId,
    ): never {
        // The OpenAPI generator extracts all metadata from these attributes
        // and produces the spec automatically
        $order = $this->orders->create($customerId, $this->requestBody());
        $this->json(['data' => $order], 201);
    }
}

// Generate the spec
$generator = new OpenApiGenerator('My Store API', '1.0.0');
$generator->scanController(OrderController::class);
$generator->scanController(ProductController::class);
file_put_contents('docs/openapi.json', $generator->toJson());
```


## Resources

- [PHP Attributes — Metadata Annotations](https://www.php.net/manual/en/language.attributes.overview.php) — Official PHP documentation for native attributes used to annotate API endpoints
- [PHP ReflectionClass](https://www.php.net/manual/en/class.reflectionclass.php) — Official PHP documentation for the Reflection API used in spec generation

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*