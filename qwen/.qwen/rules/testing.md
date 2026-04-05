# Testing Rules

## Models / Entities Testing Policy

**DO NOT** create unit tests for ORM models/entities that only contain standard configuration.

Rationale:
- The ORM is extensively tested by its maintainers
- Testing basic CRUD, relationships, and standard functionality provides no value
- Models are excluded from code coverage metrics

What NOT to test:
- Basic relationships (hasOne, hasMany, belongsTo, etc.)
- Simple CRUD operations
- Standard ORM functionality
- Factory creation without custom logic
- Basic fillable/guarded attributes, standard casting

Exceptions — What TO test:
- Custom business logic methods
- Complex accessors/mutators with business rules
- Custom scopes/query methods with specific logic
- Observer behavior and side effects

Where to test model functionality instead:
- Feature tests via HTTP endpoints and workflows
- Integration tests for model interactions
- Observer tests for event handlers
- Action/Service tests for business logic

## Framework & Tools

- **PHPUnit or Pest** — choose based on project convention
- **Mutation Testing** — verify test quality, not just coverage
- **Architectural Testing** — enforce project rules via automated tests

## Test Structure

```
tests/
├── Feature/          # Integration tests (HTTP endpoints, workflows)
├── Unit/             # Unit tests (Actions, Services, Support classes)
└── TestCase.php      # Base test case
```

## Writing Tests

```php
<?php

declare(strict_types=1);

describe('Feature Description', function (): void {
    beforeEach(function (): void {
        // Setup common test data
    });

    it('describes what it tests', function (): void {
        $result = someFunction();
        expect($result)->toBe('expected_value');
    });
});
```

### Testing Actions

```php
it('handles the action correctly', function (): void {
    $action = new YourAction();
    $result = $action->handle($request, $parameters);

    expect($result)
        ->toBeInstanceOf(ExpectedType::class);
});
```

## Test Configuration

- **Database**: Use transactions or refresh strategy for clean state
- **Environment**: Separate testing configuration
- **Coverage**: Reports in `coverage/` directory
- **Memory Limit**: Set appropriately for test suite size

## Architectural Testing

Enforced rules:
- No debugging functions (`dd()`, `dump()`, `var_dump()`) in production code
- Models must extend the base ORM model
- Classes follow naming conventions
- Enums must be proper enum classes (PHP 8.1+)
