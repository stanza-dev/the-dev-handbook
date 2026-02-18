---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-json"
---

# JSON: Document Storage

Redis JSON (RedisJSON) allows you to store, query, and manipulate JSON documents natively. It's perfect for applications that work with complex, nested data structures.

## Why Use Redis JSON?

- **Native JSON operations**: Manipulate nested structures without fetching/parsing
- **Atomic updates**: Modify specific fields without race conditions
- **JSONPath queries**: Query nested data with familiar syntax
- **Memory efficient**: Optimized binary storage format

## Basic JSON Commands

### JSON.SET - Storing Documents

```redis
# Store a JSON object
JSON.SET user:1 $ '{"name": "Alice", "age": 30, "email": "alice@example.com"}'
# Returns: OK

# Store with nested structure
JSON.SET user:2 $ '{
  "name": "Bob",
  "profile": {
    "bio": "Developer",
    "location": "NYC"
  },
  "skills": ["Python", "Redis", "Docker"]
}'
```

### JSON.GET - Retrieving Documents

```redis
# Get entire document
JSON.GET user:1
# Returns: {"name":"Alice","age":30,"email":"alice@example.com"}

# Get specific field
JSON.GET user:1 $.name
# Returns: ["Alice"]

# Get nested field
JSON.GET user:2 $.profile.location
# Returns: ["NYC"]

# Get multiple paths
JSON.GET user:2 $.name $.profile.bio
# Returns: {"$.name":["Bob"],"$.profile.bio":["Developer"]}
```

## Modifying JSON

### Updating Fields

```redis
# Update a field
JSON.SET user:1 $.age 31

# Add new field
JSON.SET user:1 $.verified true

# Update nested field
JSON.SET user:2 $.profile.location "San Francisco"
```

### Numeric Operations

```redis
# Increment number
JSON.NUMINCRBY user:1 $.age 1
# Returns: [32]

# Multiply
JSON.NUMMULTBY prices $.total 1.1    # 10% increase
```

### String Operations

```redis
# Append to string
JSON.STRAPPEND user:2 $.profile.bio " & Teacher"
# Bio is now "Developer & Teacher"

# Get string length
JSON.STRLEN user:2 $.profile.bio
# Returns: [18]
```

## Working with Arrays

```redis
# Append to array
JSON.ARRAPPEND user:2 $.skills "Kubernetes"
# Returns: [4] (new length)

# Insert at position
JSON.ARRINSERT user:2 $.skills 0 "Go"
# Inserts "Go" at the beginning

# Get array length
JSON.ARRLEN user:2 $.skills
# Returns: [5]

# Get array element by index
JSON.GET user:2 $.skills[0]
# Returns: ["Go"]

# Remove and return last element
JSON.ARRPOP user:2 $.skills
# Returns: ["Kubernetes"]

# Find index of value
JSON.ARRINDEX user:2 $.skills '"Redis"'
# Returns position of "Redis" in array
```

## JSONPath Queries

```redis
# Store sample data
JSON.SET store $ '{
  "products": [
    {"name": "Laptop", "price": 999, "inStock": true},
    {"name": "Mouse", "price": 29, "inStock": true},
    {"name": "Keyboard", "price": 79, "inStock": false}
  ]
}'

# Get all product names
JSON.GET store $.products[*].name
# Returns: ["Laptop","Mouse","Keyboard"]

# Get all prices
JSON.GET store $.products[*].price
# Returns: [999,29,79]

# Filter (products in stock)
JSON.GET store '$.products[?(@.inStock==true)].name'
# Returns: ["Laptop","Mouse"]
```

## Practical Example: Shopping Cart

```redis
# Create cart
JSON.SET cart:user:1 $ '{
  "items": [],
  "total": 0
}'

# Add item
JSON.ARRAPPEND cart:user:1 $.items '{"productId": "prod:1", "name": "Widget", "price": 29.99, "qty": 1}'

# Update quantity
JSON.SET cart:user:1 '$.items[?(@.productId=="prod:1")].qty' 3

# Calculate total (application-side typically)
JSON.GET cart:user:1 $.items[*].price

# Remove item
JSON.ARRPOP cart:user:1 $.items 0
```

📖 [Redis JSON Documentation](https://redis.io/docs/latest/develop/data-types/json/)

## Resources

- [Redis JSON](https://redis.io/docs/latest/develop/data-types/json/) — Complete guide to Redis JSON

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*