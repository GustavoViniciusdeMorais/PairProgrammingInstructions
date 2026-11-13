---
name: php-standards
description: 'PHP coding standards for this Laravel 10 project. Use when: writing or reviewing PHP classes, methods, properties, enums, strings, ternaries, if statements, loops, routing, validation, or constructors. Triggers: "check code style", "review PHP", "does this follow standards", "PSR-12", "coding standards", "code quality".'
---

# PHP Coding Standards

This project follows [PSR-12](https://www.php-fig.org/psr/psr-12/) with the conventions below.

## Classes

- Do **not** use `final` by default.

## Nullable & Union Types

Use short nullable notation instead of union with `null`.

```php
public ?string $variable;
```

## Void Return Types

Always declare `void` when a method returns nothing.

```php
public function scopeArchived(Builder $query): void
{
    $query->...
}
```

## Typed Properties

Always declare property types. Do not use docblocks for this.

```php
class Foo
{
    public string $bar;
}
```

## Enums

Use camelCase for enum values.

```php
enum Suit: string {
    case clubs = 'clubs';
    case diamonds = 'diamonds';
    case hearts = 'hearts';
    case spades = 'spades';
}
```

## Docblocks

Use simple docblocks with `@param` and `@return` only.

```php
/** @return Collection<int, SomeObject> */
function someFunction(): Collection {}
```

## Constructors (Property Promotion)

Promote all properties when possible. Place each on its own line with a trailing comma.

```php
class MyClass {
    public function __construct(
        protected string $firstArgument,
        protected string $secondArgument,
    ) {}
}
```

## Traits

Apply each trait on its own line with its own `use` keyword.

```php
class MyClass
{
    use TraitA;
    use TraitB;
}
```

## Strings

Prefer string interpolation over `sprintf` or `.` concatenation.

```php
$greeting = "Hi, I am {$name}.";
```

## Ternary Operators

Short ternaries can stay on one line. Long ones: one portion per line.

```php
$name = $isFoo ? 'foo' : 'bar';

$result = $object instanceof Model ?
    $object->name :
    'A default value';
```

## If Statements

- Always use curly braces.
- Unhappy path first (early return), happy path last (unindented).
- Avoid `else`; prefer ternary.
- Aggregate compound conditions to reduce cognitive complexity.

```php
if (! $goodCondition) {
    throw new Exception;
}

if (
    $conditionA
    && $conditionB
    && $conditionC
) {
    // do stuff
}
```

## Loops

Prefer Laravel Collections over `foreach`.

```php
$users->filter(fn(User $user): bool => $user->is_active)
    ->each(function (User $user) {
        $user->update(['attribute' => 'test']);
    });
```

## Whitespace

Add blank lines between statements unless they are a sequence of equivalent single-line operations.

## Routing

URLs use kebab-case. Use route tuple notation.

```php
Route::get('open-source', [OpenSourceController::class, 'index'])->name('open-source');
Route::get('news/{newsItem}', [NewsItemsController::class, 'index']);
```

## Validation

Always use array notation for multiple rules.

```php
public function rules(): array
{
    return [
        'email' => ['required', 'email'],
    ];
}
```

Use `data_get()` for safe access to array keys or object attributes.

```php
data_get($user, 'email', null);
```

Use Laravel `Validator` facade when validating outside a FormRequest.

```php
$validated = Validator::validate($payload, [
    'name' => ['nullable', 'string'],
    'email' => ['nullable', 'string'],
]);
```
---

## References
- [PHP Coding Standards](../../php_coding_standards.md)