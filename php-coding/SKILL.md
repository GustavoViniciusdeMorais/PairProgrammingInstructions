---
name: php-coding
description: 'PHP/Laravel coding skill for this project. Use when creating or reviewing Controllers, Actions, DTOs, Models, Services, Migrations, QueryBuilders, or any PHP class in this Laravel 10 codebase. Triggers: "create a controller", "create an action", "create a DTO", "write a migration", "review this PHP class", "refactor this method", "add a route", "create a service", "write a test", "review coding standards".'
argument-hint: 'Describe what you want to create or review (e.g., "CreateUserAction", "UserController store method")'
---

# PHP Coding Skill — Laravel 10 / PHP 8

## Mandatory: Follow the Agent Rules

Always read and follow [`../../copilot-instructions.md`](../../copilot-instructions.md) before creating, refactoring, or reviewing any PHP code.

Concretely, this covers the agent rules on: no comments in code, database messages in Brazilian Portuguese, transaction control for multi-table updates, `vscode_askQuestions`-driven pair programming, building only the main class/function structure, and never running terminal commands.

## When to Use
- Creating or refactoring Controllers, Actions, DTOs, Models, Resources, Migrations
- Reviewing PHP code for standards compliance
- Writing Eloquent queries, QueryBuilder filters, or DB transactions
- Implementing business logic with Lorisleiva Actions
- Creating Spatie Laravel Data DTOs with validation
- Writing Pest tests for new features

---

## Stack & Libraries

| Concern | Library |
|---|---|
| Framework | Laravel 10 |
| PHP | 8.x |
| DTOs | `spatie/laravel-data` |
| Actions | `lorisleiva/laravel-actions` |
| QueryBuilder | `spatie/laravel-query-builder` |
| ORM | Eloquent |
| Testing | Pest |

---

## Coding Constraints (Enforce Always)

- Max **2 `if` statements** per function
- Max **1 `return`** per function
- Max **3 parameters** per function — use a DTO if exceeded
- Max **20 methods** per class
- Max **200 lines** per class
- Functions must **not** alter variables outside their scope
- Use **early returns** (happy path last)
- Use **type hinting** and **return types** on all methods
- Use **Laravel Collections** instead of `foreach` loops
- Use **PHP helper functions** (`strlen`, `implode`, etc.) instead of reimplementing them
- Apply **SOLID, DRY, KISS** principles
- Correct all **spelling mistakes** in variables, methods, classes, and strings
- DB messages stored (e.g., error translations) must be in **Brazilian Portuguese**
- **No inline comments** unless logic is genuinely complex

---

## Architecture Patterns

The layer contracts, class templates and the request flow are the single source of truth in the [PHP Architecture](../php-architecture/SKILL.md) skill. Summary of ownership:

| Layer | Owns | Never owns |
|---|---|---|
| Controller | HTTP boundary, status code, envelope, logging | business rules, queries, formatting |
| DTO | input shape + validation rules | persistence, HTTP |
| Action | use case, transactions, jobs, queries | HTTP responses |
| Model / QueryBuilder | relations, casts, scopes, reads | HTTP, request input |
| Resource | single-record output shape | secrets, raw internal JSON |
| Collection | paginated shape + `PaginationResource` | business rules |

Folder convention: `src/Actions/<Module>/*Action.php`, `src/Data/<Module>/*Data.php`, `src/Http/Resources/<Module>/Resource/*Resource.php` and `src/Http/Resources/<Module>/Collection/*Collection.php`.

Load the [PHP Architecture](../php-architecture/SKILL.md) skill for the code templates, the Model → DTO → Action → Controller → Resource → Collection → Route flow, and the transaction rules.

---

## Eloquent Best Practices

- Eager load relationships to avoid N+1 (`with()`)
- Use query scopes for reusable filters
- Use `casts` for JSON fields and booleans
- Write queries on multiple lines for readability:

```php
User::query()
    ->where('active', true)
    ->whereHas('roles', fn ($q) => $q->where('name', 'admin'))
    ->orderBy('name')
    ->get();
```

---

## PHP Style (PSR-12 + Project Conventions)

- Nullable shorthand: `?string` not `string|null`
- `void` return type for methods that return nothing
- Typed properties — no docblock for simple types
- Enum values in camelCase
- Property promotion in constructors (comma after last item)
- String interpolation preferred: `"Hi, {$name}"`
- Each trait on its own `use` line
- Ternary on one line for short expressions; multi-line for long ones
- Avoid `else` — use early return or ternary
- Compound `if` conditions stacked vertically with `&&`
- Always use curly braces in `if` statements

---

## Security Checklist (OWASP)

- Never concatenate raw user input into queries — use Eloquent ORM or query bindings
- Validate all inputs via FormRequest or DTO `rules()`
- Return only necessary fields in API Resources
- Never expose stack traces in production responses
- Use `trans()` for user-facing messages (not hardcoded strings)

---

## Procedure: Creating a New Feature

Follow the layered procedure in the [PHP Architecture](../php-architecture/SKILL.md) skill: Model → Migration → DTO → Action → Controller → Resource → Collection → Route → Test.

Controller response flow: `Model` → `Action` (QueryBuilder/ORM) → `Collection` (wraps `Resource`) → JSON.

The Resource and Collection layers are defined by the project architect and are never optional.

---

## References
- [Agent Instructions](../../copilot-instructions.md) — **mandatory, always apply first**
- [PHP Architecture](../php-architecture/SKILL.md) — layer contracts, templates and request flow
- [PHP Coding Standards](../../php_coding_standards.md)
- [Task Report Instructions](../../task-report-instructions.md)
