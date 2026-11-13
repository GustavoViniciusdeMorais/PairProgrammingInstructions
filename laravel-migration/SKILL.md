---
name: laravel-migration
description: 'Laravel migration best practices. Use when: creating a migration, adding a column, modifying a table, writing up() or down() methods, reviewing a migration file. Triggers: "create migration", "add column", "new migration", "write migration", "migration best practices".'
argument-hint: 'Describe the migration you want to create'
---

# Laravel Migration Best Practices

> **Database engine: MySQL 8.0.** All migrations are written for MySQL 8.0. Features specific to other engines (e.g., SQLite, PostgreSQL) do not apply here.

Before writing any migration, ask the user the following questions. Ask them all at once, numbered, in a single message.

---

## Questions to Ask the User

Ask these questions before generating any code:

1. **Column position** — After which existing column should the new column be placed? (Used with `->after('column_name')`. If not provided, it will be appended at the end of the table.)

2. **Nullable or required** — Should this column accept `NULL` values, or is it required (`NOT NULL`)? If required, is there a safe default value for existing rows?

3. **Default value** — Should this column have a default value? If yes, what is it?

4. **Index** — Should this column be indexed? Choose one: `none`, `index`, `unique`, or `primary`.

5. **Foreign key** — Is this column a foreign key? If yes, which table and column does it reference, and what should happen on delete (`cascade`, `set null`, `restrict`)?

6. **Column comment** — Should a human-readable description/comment be added to this column in the database? If yes, what should it say?

7. **Reversible migration** — Does the `down()` method need to fully reverse this migration? Confirm what the rollback should do (e.g., drop column, restore old type).

---

## After Collecting Answers

Apply the following rules when writing the migration:

- Use `->after('column')` to position the column explicitly (MySQL/MariaDB only — skip for SQLite/PostgreSQL)
- Use `->nullable()` only when confirmed; otherwise default to `NOT NULL`
- Use `->default(value)` when a default is provided
- Use `->index()`, `->unique()`, or leave clean based on the index answer
- Add `->constrained('table')->onDelete('action')` for foreign keys using `foreignId()` or `foreign()`
- Use `->comment('description')` when a column comment is provided
- Always implement a proper `down()` that reverses the `up()` exactly
- Wrap `Schema::table` changes (not `Schema::create`) with existence checks using `Schema::hasColumn()` when modifying existing columns
- Use `$table->unsignedBigInteger()` or `$table->foreignId()` for foreign key columns — never plain `integer()`
