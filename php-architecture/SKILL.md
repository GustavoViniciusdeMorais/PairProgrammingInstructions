---
name: php-architecture
description: 'Action / Domain / Response layered architecture for this Laravel 10 API. Use when: designing or reviewing a feature end to end, deciding which layer owns a responsibility, placing validation, business rules, persistence or formatting, wiring a Controller to an Action, building Resources and Collections, choosing between Action, Service, Repository or Model scope, converting fat controllers into Actions, or checking that the Model → DTO → Action → Controller → Resource → Collection → Route flow is complete. Triggers: "php architecture", "action domain response", "layer responsibility", "which layer", "create a feature", "split this controller", "add an endpoint", "response contract", "request flow".'
argument-hint: 'Describe the feature or class you want to design or review (e.g., "CreateFormAction endpoint", "review GetPagesAction layers")'
---

# PHP Architecture — Action / Domain / Response

## Mandatory: Follow the Agent Rules

Always read and follow [`../../copilot-instructions.md`](../../copilot-instructions.md) before creating, refactoring, or reviewing any PHP code. The agent rules are the source of truth for coding constraints, transactions, logging, messages and pair-programming behavior.

---

## When to Use

- Designing a new API feature from scratch (which files, which order)
- Reviewing an existing feature for missing layers or misplaced responsibility
- Deciding where validation, business rules, persistence or formatting belongs
- Splitting a fat controller, a fat Action or a Model with business logic
- Verifying the endpoint contract (status / message / data)

Not for: writing a single isolated migration (use `laravel-migration`), or PSR-12 style details alone (use `php-standards`).

---

## Stack & Libraries

| Concern | Library |
|---|---|
| Framework | Laravel 10 / PHP 8 |
| DTOs | `spatie/laravel-data` |
| Actions | `lorisleiva/laravel-actions` |
| QueryBuilder | `spatie/laravel-query-builder` |
| ORM | Eloquent |
| Testing | Pest |

---

## The Three Rings

The architecture has three concentric responsibilities:

| Ring | Question it answers | Lives in |
|---|---|---|
| **Action** | What does the system do? (use case) | `src/Actions/<Module>/*Action.php` |
| **Domain** | What is true and how is data stored? | `src/Models`, `src/Data` (DTOs), QueryBuilders, scopes |
| **Response** | What does the client see? | `src/Http/Resources/<Module>/Resource/*Resource.php`, `.../Collection/*Collection.php` |

The Controller is HTTP plumbing only: it owns no rule, no query and no formatting.

### Responsibility Matrix

| Responsibility | Model | DTO | Action | Controller | Resource | Collection |
|---|---|---|---|---|---|---|
| Table mapping, casts, relations, scopes | ✅ | | | | | |
| Input shape + validation rules | | ✅ | | | | |
| Use case orchestration | | | ✅ | | | |
| Transactions across tables | | | ✅ | | | |
| Dispatch jobs / events | | | ✅ | | | |
| Auth extraction (`auth()`) | | | | ✅ | | |
| HTTP status + envelope | | | | ✅ | | |
| Single-record output shape | | | | | ✅ | |
| Pagination + collection shape | | | | | | ✅ |

---

## Layer Contracts

### Model (Domain)

- Eloquent model, one table, `$table`, `$guarded` or `$fillable`, `$casts` for JSON/boolean.
- Relationships, accessors/mutators and query scopes belong here.
- No HTTP, no request input, no JSON response formatting.
- Prefer `data_get()` for safe attribute reads.

### DTO (Domain input contract)

- Spatie `Data` class with promoted `public` properties.
- `rules(ValidationContext $context)` returns the rules; `messages(...$args)` returns PT-BR messages.
- Field with multiple rules always uses array notation.
- Never validate in the controller. The Action receives an already valid DTO.
- If a method would need more than 3 parameters, replace them with a DTO.

```php
class PostData extends Data
{
    public function __construct(
        public string $title,
        public string $content,
    ) {
    }

    public static function rules(ValidationContext $context): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'content' => ['required', 'string'],
        ];
    }

    public static function messages(...$args): array
    {
        return [
            'title.required' => 'O título é obrigatório.',
            'content.required' => 'O conteúdo é obrigatório.',
        ];
    }
}
```

### Action (Use case)

- Lorisleiva `AsAction`, one public `handle()`, invoked with `ActionClass::run(...)`.
- All business rules, transactions, dispatch calls and multi-table writes live here.
- Returns a Model, a Collection, a QueryBuilder, or `void` — never a JSON response.
- A list Action returns a `QueryBuilder` so the controller decides pagination.

```php
class ExampleAction
{
    use AsAction;

    public function handle(PostData $data): void
    {
    }
}
```

### Controller (HTTP boundary)

- Thin: read the request, call the Action, wrap the result in a Resource/Collection, return JSON.
- `try/catch (Throwable $e)`, log with the `[system-feature-name]` prefix, return the standard envelope.
- Use `findOrFail` for single-record lookups.
- Use `->paginate($request->get('per_page', 15))` on the Action's QueryBuilder.
- Bulk endpoints accept a single ID or a list of IDs.

```php
class ExampleController extends Controller
{
    public function someFunctionName($id, Request $request)
    {
        try {
            $user = User::findOrFail($id);

            return response()->json([
                "status" => "success",
                "message" => "Operação realizada com sucesso.",
                "data" => [$user]
            ], 200);
        } catch (Throwable $e) {
            Log::error('[system-feature-name] - Erro Title.', [
                'message' => $e->getMessage(),
                'file' => $e->getFile(),
                'line' => $e->getLine(),
                'details' => $e->getTraceAsString(),
            ]);

            return response()->json([
                "status" => "error",
                "message" => trans("system.default.error"),
                "data" => []
            ], 500);
        }
    }
}
```

### Resource (Response, single record)

- `JsonResource`, `toArray($request): array`.
- Return only the necessary fields — never tokens, secrets or internal JSON blobs.
- Use `whenLoaded()` for relations, `optional()->format('Y-m-d H:i:s')` for dates.
- Keep helpers small and protected.

```php
class ProductResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
        ];
    }
}
```

### Collection (Response, paginated list)

- `ResourceCollection`, set `public $collects = ProductResource::class;`.
- Override `toResponse($request)` and merge `PaginationResource`, which adds the `meta` and `links` blocks.
- Ask me about the class path of `PaginationResource` with `vscode_askQuestions`.

```php
use Custom\Http\Resources\PaginationResource;
use Illuminate\Http\Resources\Json\ResourceCollection;
use Illuminate\Http\Request;

class ProductCollection extends ResourceCollection
{
    public $collects = ProductResource::class;

    public function toResponse($request)
    {
        return array_merge(
            ['data' => $this->collection],
            (new PaginationResource($this->resource))->toArray($request)
        );
    }
}
```

### QueryBuilder (Domain read contract)

- Spatie `QueryBuilder` inside the list Action: `allowedFields`, `allowedFilters`, `allowedIncludes`, `allowedSorts`, `defaultSort`.
- Controller usage: `return new ProductCollection(GetProductsAction::run()->paginate($request->get('per_page', 15)));`

```php
$users = QueryBuilder::for(User::class)
    ->allowedFields(['id', 'name'])
    ->allowedFilters('name')
    ->allowedIncludes(['posts'])
    ->allowedSorts(['name'])
    ->defaultSort('name');
```

---

## Request Flow (Model → Action → Response → JSON)

```
Route
  ↓
Controller            (HTTP: auth, status code, envelope, logging)
  ↓
DTO                   (validation + typed input shape)
  ↓
Action                (use case: rules, transaction, jobs, queries)
  ↓
Model / QueryBuilder  (Domain: Eloquent, relations, scopes)
  ↓
Resource              (Response: one record)
  ↓
Collection            (Response: paginator + PaginationResource)
  ↓
JSON                  (status / message / data)
```

Invariants of the flow:

1. Only the Controller knows about HTTP.
2. Only the DTO validates input.
3. Only the Action opens a transaction.
4. Only the Response ring formats output.
5. The Domain ring never imports `Request` or `JsonResponse`.

---

## Transactions (Domain side, inside the Action)

Use transactions whenever more than one table is written. Prefer the closure form and keep side updates inside it.

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () use ($data) {
    $product = $this->createProduct($data);
    $this->createRelations($product);
});
```

Do not write and read the same record from the DB in the same step — work with the ORM object returned by `save()`/`create()`. `DB::transaction()` returns the closure's return value, so capture created models there instead of re-querying.

---

## Constraint Enforcement Checklist

Run this on every new or refactored class:

- [ ] ≤ 2 `if` statements per function
- [ ] ≤ 1 `return` per function
- [ ] ≤ 3 parameters per function (else a DTO)
- [ ] ≤ 20 methods per class
- [ ] ≤ 200 lines per class
- [ ] No function mutates a variable outside its own scope
- [ ] Type hints and return types on every method
- [ ] Collections used instead of `foreach`
- [ ] PHP helper functions used (`strlen`, `implode`)
- [ ] Laravel helper functions used (`data_get`)
- [ ] Early return — happy path last, no `else`
- [ ] Curly braces on every `if`, compound conditions stacked vertically
- [ ] No N+1 — eager load with `with()`
- [ ] Spelling corrected in variables, methods, classes and strings
- [ ] No inline comments unless the logic is genuinely complex
- [ ] DB-registered messages in Brazilian Portuguese
- [ ] Input validated in the DTO, never concatenated into SQL
- [ ] Resource never exposes tokens, passwords or raw internal JSON
- [ ] Multi-table write wrapped in a transaction
- [ ] Controller catches `Throwable` and returns status / message / data

---

## Procedure: Creating a New Feature

1. **Model** — connect the table, define casts, relations and scopes.
2. **Migration** — only if the table or column must change.
3. **DTO** — input shape with `rules()` and PT-BR `messages()`.
4. **Action** — the use case: validation handoff, transaction, queries, jobs.
5. **Controller** — thin method: call the Action, wrap the result, return JSON.
6. **Resource** — required layer, one record.
7. **Collection** — required layer, paginated output + `PaginationResource`.
8. **Route** — register in `routes/api.php` with kebab-case URLs.
9. **Test** — Pest: happy path, error cases, edge cases.

The Resource and Collection layers are defined by the project architect and are never optional.

---

## Smells and Their Fix

| Smell | Fix |
|---|---|
| Controller contains an `if` about business data | Move the rule into the Action |
| Action returns `response()->json(...)` | Return the model/collection; let the Controller respond |
| Action validates with `$request->validate()` | Move the rules into the DTO |
| Controller builds an array by hand | Add or extend the Resource |
| Model has HTTP or formatting logic | Move it out to the Response ring |
| Action updates several tables without a transaction | Wrap in `DB::transaction()` |
| Endpoint handles only one ID at a time | Support a single ID or a list of IDs |
| Resource dumps the whole model | List the fields explicitly |

---

## References

- [Agent Instructions](../../copilot-instructions.md) — mandatory, always apply first
- [PHP Coding Skill](../php-coding/SKILL.md) — templates and per-layer detail
- [PHP Coding Standards](../../php_coding_standards.md)
- [PHP Standards Skill](../php-standards/SKILL.md)
