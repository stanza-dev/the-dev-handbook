---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-repository-pattern"
---

# Repository Pattern

## Introduction
The Repository pattern abstracts data access logic, providing a collection-like interface for domain objects. It separates business logic from data access details.

## Key Concepts
- **Repository Interface**: Defines collection-like methods (find, save, delete) for domain objects.
- **Concrete Repository**: Implements the interface using a specific data source (MySQL, MongoDB, API).
- **In-Memory Repository**: A testing implementation that stores data in arrays.
- **Separation of Concerns**: Business logic never touches database queries directly.

## Real World Context
Your application starts with MySQL but needs to support PostgreSQL for some clients and an external API for others. The Repository pattern means your services work against an interface. Swapping MySQL for PostgreSQL requires only a new repository class, not changes to business logic.

## Deep Dive

### Basic Repository

```php
<?php
interface UserRepositoryInterface {
    public function find(int $id): ?User;
    public function findByEmail(string $email): ?User;
    public function findAll(): array;
    public function save(User $user): void;
    public function delete(User $user): void;
}

class MySQLUserRepository implements UserRepositoryInterface {
    public function __construct(private PDO $pdo) {}

    public function find(int $id): ?User {
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
        $stmt->execute(['id' => $id]);
        $row = $stmt->fetch();
        return $row ? $this->hydrate($row) : null;
    }

    public function findByEmail(string $email): ?User {
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE email = :email');
        $stmt->execute(['email' => $email]);
        $row = $stmt->fetch();
        return $row ? $this->hydrate($row) : null;
    }

    public function findAll(): array {
        $stmt = $this->pdo->query('SELECT * FROM users');
        return array_map([$this, 'hydrate'], $stmt->fetchAll());
    }

    public function save(User $user): void {
        if ($user->id === null) {
            $this->insert($user);
        } else {
            $this->update($user);
        }
    }

    public function delete(User $user): void {
        $stmt = $this->pdo->prepare('DELETE FROM users WHERE id = :id');
        $stmt->execute(['id' => $user->id]);
    }

    private function hydrate(array $row): User {
        return new User(
            id: (int) $row['id'],
            name: $row['name'],
            email: $row['email']
        );
    }

    private function insert(User $user): void {
        $stmt = $this->pdo->prepare(
            'INSERT INTO users (name, email) VALUES (:name, :email)'
        );
        $stmt->execute(['name' => $user->name, 'email' => $user->email]);
        $user->id = (int) $this->pdo->lastInsertId();
    }

    private function update(User $user): void {
        $stmt = $this->pdo->prepare(
            'UPDATE users SET name = :name, email = :email WHERE id = :id'
        );
        $stmt->execute(['id' => $user->id, 'name' => $user->name, 'email' => $user->email]);
    }
}
```

All SQL is encapsulated in the repository. Business logic never sees a query.

### In-Memory Repository (For Testing)

```php
<?php
class InMemoryUserRepository implements UserRepositoryInterface {
    private array $users = [];
    private int $nextId = 1;

    public function find(int $id): ?User {
        return $this->users[$id] ?? null;
    }

    public function findByEmail(string $email): ?User {
        foreach ($this->users as $user) {
            if ($user->email === $email) return $user;
        }
        return null;
    }

    public function findAll(): array {
        return array_values($this->users);
    }

    public function save(User $user): void {
        if ($user->id === null) $user->id = $this->nextId++;
        $this->users[$user->id] = $user;
    }

    public function delete(User $user): void {
        unset($this->users[$user->id]);
    }
}
```

Tests use InMemoryUserRepository for speed and isolation without database setup.

### Service Using Repository

```php
<?php
class UserService {
    public function __construct(
        private UserRepositoryInterface $repository
    ) {}

    public function register(string $name, string $email, string $password): User {
        if ($this->repository->findByEmail($email)) {
            throw new DomainException('Email already registered');
        }
        $user = new User(
            name: $name,
            email: $email,
            passwordHash: password_hash($password, PASSWORD_DEFAULT)
        );
        $this->repository->save($user);
        return $user;
    }
}

// Production
$service = new UserService(new MySQLUserRepository($pdo));
// Testing
$service = new UserService(new InMemoryUserRepository());
```

The service works identically with any repository implementation.

## Common Pitfalls
1. **Leaking query logic** — Do not pass raw SQL or query builders through the repository interface. Keep it domain-focused.
2. **Anemic repositories** — Repositories with only CRUD methods miss the point. Add domain-specific finders like `findActiveByRole()`.

## Best Practices
1. **Name methods after domain concepts** — Use `findActiveUsers()` instead of `findByStatusAndDate()`.
2. **Always create an in-memory implementation** — It forces a clean interface and enables fast tests.

## Summary
- Repository pattern abstracts data access behind a collection-like interface.
- Services depend on the repository interface, not the concrete implementation.
- In-memory repositories enable fast, isolated testing.
- Name repository methods after domain concepts, not database operations.

## Code Examples

**Repository pattern with in-memory implementation for testing**

```php
<?php
declare(strict_types=1);

interface ProductRepository {
    public function find(int $id): ?Product;
    public function findByCategory(string $category): array;
    public function save(Product $product): void;
}

class InMemoryProductRepository implements ProductRepository {
    private array $products = [];
    private int $nextId = 1;

    public function find(int $id): ?Product {
        return $this->products[$id] ?? null;
    }

    public function findByCategory(string $category): array {
        return array_filter(
            $this->products,
            fn(Product $p) => $p->category === $category
        );
    }

    public function save(Product $product): void {
        if ($product->id === null) {
            $product->id = $this->nextId++;
        }
        $this->products[$product->id] = $product;
    }
}

class ProductService {
    public function __construct(
        private ProductRepository $repository
    ) {}

    public function createProduct(string $name, string $category, float $price): Product {
        $product = new Product(name: $name, category: $category, price: $price);
        $this->repository->save($product);
        return $product;
    }
}

// Testing with in-memory implementation
$repo = new InMemoryProductRepository();
$service = new ProductService($repo);
$product = $service->createProduct("Widget", "tools", 9.99);
echo $repo->find($product->id)->name;
?>
```


## Resources

- [Repository Pattern](https://martinfowler.com/eaaCatalog/repository.html) — Martin Fowler on Repository pattern

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*