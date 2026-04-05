# PHP Code Style

## Strict PHP

- All PHP files must declare `declare(strict_types=1)`
- Full type hints required for all parameters and return types
- Use `===` instead of `==` (strict comparisons)
- Use modern PHP 8.x features and proper type casting
- Trailing commas in multiline arrays and parameter lists

## Class Organization

Specific order for class elements:
1. Constants
2. Properties
3. Constructor
4. Public methods
5. Protected methods
6. Private methods

## Naming Conventions

- Classes: PascalCase (`UserService`, `OrderProcessor`)
- Methods: camelCase (`handlePayment()`, `isValid()`)
- Variables: camelCase (`$userList`, `$totalCount`)
- Constants: UPPER_SNAKE_CASE (`MAX_RETRIES`, `API_TIMEOUT`)
- Interfaces: Prefix with `I` or suffix with `able`/`Interface` (`Cacheable`, `RepositoryInterface`)
- Traits: PascalCase with `Trait` suffix (`LoggableTrait`, `TimestampableTrait`)

## Code Quality Tools

| Tool | Purpose | Config |
|------|---------|--------|
| PHP-CS-Fixer / Pint | Code formatting | PSR-12 with strict rules |
| PHPStan (Level 7+) | Static analysis | Strict typing enabled |
| Rector | Code modernization | Latest PHP version ruleset |
| Cognitive Complexity | Complexity limits | class: 85, function: 8 |
