# PHP Coding Standards

The code style must follow [PSR-12](https://www.php-fig.org/psr/psr-12/). 

### Class Standards

By default, we don't use `final`. In our team, there aren't many benefits that `final` offers.

### Nullable and union types

Whenever possible, use the short nullable notation for a type, instead of using a union of the type with `null`.

**Prefer**

```php
public ?string $variable;
```

### Void return types

If a method returns nothing, it should be indicated with `void`.
This makes it clearer to users of your code what your intention was when writing it.

**Prefer**

```php
// in a Laravel model
public function scopeArchived(Builder $query): void
{
    $query->
        ...
}
```

## Typed Properties

You should define the type of a property whenever possible. Do not use a docblock.

**Prefer**
```php
class Foo
{
    public string $bar;
}
```

## Enums

Enum values should use camelCase.

```php
enum Suit:string {  
    case clubs='clubs';
    case diamonds='diamonds';
    case hearts='hearts';
    case spades='spades';
}

Suit::diamonds;
```

## Docblocks

Use simple docblocks.

**Prefer**
```php
use Url;

/**
* @param string $url
* @return Url
*/
function fromString(string $url): Url
{
    // ...
}
```

```php
use \Illuminate\Support\Collection

/** @return Collection<int,SomeObject> */
function someFunction(): Collection
{
    //
}
```

## Constructors (Property Promotion)

Use property promotion in the constructor if all properties can be promoted. To make it readable, place each one on its own line. Use a comma after the last one.

**Prefer**
```php
class MyClass {
    public function __construct(
        protected string $firstArgument,
        protected string $secondArgument,
    ) {}
}
```

## Traits

Each trait must be applied on its own line, and the `use` keyword must be used for each of them. This will result in clean diffs when traits are added or removed.

**Prefer**
```php
class MyClass
{
    use TraitA;
    use TraitB;
}
```

## Strings

Prefer string interpolation instead of `sprintf` and the `.` operator.

**Prefer**
```php
$greeting = "Hi, I am {$name}.";
```

## Ternary Operators

Each portion of a ternary expression should be on its own line, unless it's a very short expression.

**Prefer**
```php
$name = $isFoo ? 'foo' : 'bar';
```

**Prefer**

```php
$result = $object instanceof Model ?
    $object->name :
   'A default value';
```

## If statements

### Curly Braces

Always use curly braces.

**Prefer**
```php
if ($condition) {
   ...
}
```

### Happy Path

Generally, a function should have its unhappy path first and its happy path last. 
In most cases, this will cause the happy path to be in an unindented part of the function, which makes it more readable.

**Prefer**
```php
if (! $goodCondition) {
  throw new Exception;
}
```

**Avoid**
```php
if ($goodCondition) {
 // do something
}

throw new Exception;
```

### Avoid else

**Prefer**
```php
$condition
    ? $this->doSomething()
    : $this->doSomethingElse();
```

### Compound ifs

If possible aggregate ifs to avoid cognitive complexity.

**Prefer**
```php
if (
    $conditionA
    && $conditionB
    && $conditionC
) {
  // do stuff
}
```
**Other option**
```php
return
    DB::table('users')->where('id', $id)->first()
    ?? new \stdClass();
```

## Loops

**Prefer**
```php
$users->filter(fn(User $user): bool => $user->is_active)
    ->each(function (User $user) {
        $user->update(['attribute' => 'test']);
    });

collect($data$)->each(function ($item) {
    // code..
});
```

## Whitespace

Statements should be allowed to breathe. In general, always add blank lines between statements, unless they are a sequence of equivalent single-line operations. This isn't something executable, it's a matter of what looks best in your context.

**Prefer**
```php
public function getPage($url)
{
    $page = $this->pages()->where('slug', $url)->first();

    if (! $page) {
        return null;
    }

    if ($page['private'] && ! Auth::check()) {
        return null;
    }

    return $page;
}
```

## Routing

Urls should use kebab-case.
Prefer using route tuple notation.

**Prefer**
```php
Route::get('/', [HomeController::class, 'index']);

Route::get('open-source', [OpenSourceController::class, 'index']);

Route::get('open-source', [OpenSourceController::class, 'index'])->name('open-source');

Route::get('news/{newsItem}', [NewsItemsController::class, 'index']);

```

## Validation

When using multiple rules for a field always use array notation.

**Prefer**
```php
public function rules()
{
    return [
        'email' => ['required', 'email'],
    ];
}
```

**data get in laravel**
User to safe get array keys or object attributes
```php
data_get($user, 'email', null);
```

**Laravel Validator**
```php
use Illuminate\Support\Facades\Validator;

$validated = Validator::validate($payload, [
    'name' => ['nullable', 'string'],
    'email' => ['nullable', 'string'],
    'data' => ['nullable', 'array'],
    'data.files' => ['nullable', 'array'],
]);

$validated['data'] = [...$user->data, ...data_get($validated, 'data', [])];

$user->update($validated);
```
