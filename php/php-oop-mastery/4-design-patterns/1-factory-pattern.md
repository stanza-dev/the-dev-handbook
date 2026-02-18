---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-factory-pattern"
---

# Factory Pattern

The Factory pattern creates objects without specifying their exact class. It centralizes object creation logic.

## Simple Factory

```php
<?php
interface Logger {
    public function log(string $message): void;
}

class FileLogger implements Logger {
    public function log(string $message): void {
        file_put_contents('app.log', $message . "\n", FILE_APPEND);
    }
}

class DatabaseLogger implements Logger {
    public function log(string $message): void {
        // Save to database
    }
}

class ConsoleLogger implements Logger {
    public function log(string $message): void {
        echo "[LOG] $message\n";
    }
}

class LoggerFactory {
    public static function create(string $type): Logger {
        return match($type) {
            'file' => new FileLogger(),
            'database' => new DatabaseLogger(),
            'console' => new ConsoleLogger(),
            default => throw new InvalidArgumentException("Unknown logger: $type"),
        };
    }
}

// Usage
$logger = LoggerFactory::create('file');
$logger->log('Application started');
```

## Factory Method Pattern

```php
<?php
abstract class Document {
    abstract public function createPages(): array;
    
    public function render(): string {
        $output = '';
        foreach ($this->createPages() as $page) {
            $output .= $page->render();
        }
        return $output;
    }
}

class Resume extends Document {
    public function createPages(): array {
        return [
            new SkillsPage(),
            new ExperiencePage(),
            new EducationPage(),
        ];
    }
}

class Report extends Document {
    public function createPages(): array {
        return [
            new IntroductionPage(),
            new ResultsPage(),
            new ConclusionPage(),
        ];
    }
}
```

## Abstract Factory

```php
<?php
interface Button {
    public function render(): string;
}

interface Checkbox {
    public function render(): string;
}

// Abstract Factory
interface UIFactory {
    public function createButton(): Button;
    public function createCheckbox(): Checkbox;
}

// Concrete implementations
class DarkButton implements Button {
    public function render(): string {
        return '<button class="dark">Click</button>';
    }
}

class DarkCheckbox implements Checkbox {
    public function render(): string {
        return '<input type="checkbox" class="dark">';
    }
}

class DarkUIFactory implements UIFactory {
    public function createButton(): Button {
        return new DarkButton();
    }
    
    public function createCheckbox(): Checkbox {
        return new DarkCheckbox();
    }
}

// Client code works with any factory
function renderForm(UIFactory $factory): string {
    $button = $factory->createButton();
    $checkbox = $factory->createCheckbox();
    
    return $checkbox->render() . $button->render();
}
```

## Resources

- [Factory Pattern](https://refactoring.guru/design-patterns/factory-method/php/example) — Factory pattern examples and explanations

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*