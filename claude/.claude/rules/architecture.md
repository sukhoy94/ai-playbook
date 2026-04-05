# Architecture Patterns

## Business Logic

- **Action Pattern** — all business logic in dedicated Action classes with a single `handle()` method
- **Service Layer**: implemented via Action classes (no separate service classes unless cross-cutting)
- **Repository Pattern**: use only when abstracting multiple data sources; otherwise work directly with models/ORM

## Database

- Every DB structure change → new migration
- Every DB data change → update seeder + factory
- Prefer ORM models over raw queries
- Prefer ORM relationships over manual joins
- Prefer eager loading over lazy loading (N+1 prevention)
- Use database-level constraints in addition to application validation

## Performance

- Use caching strategically (Redis/Memcached for shared cache)
- Database indexing on frequently queried columns
- Queue long-running tasks (emails, imports, external API calls)
- Profile before optimizing — measure, don't guess

## Error Handling

- Use custom exception classes for domain-specific errors
- Never swallow exceptions — log and rethrow or handle explicitly
- Return consistent error shapes in APIs: `{ error: string, code: number }`
- Validate all inputs at function/method boundaries

## Development Tools

- Static analysis (PHPStan) in CI pipeline
- Code formatting enforced by pre-commit hook
- Debugging tools available in development environment
- IDE helpers for autocompletion on dynamic code
