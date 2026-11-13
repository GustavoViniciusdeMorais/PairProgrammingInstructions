---
name: php-tinker
description: 'Runs one-shot PHP validation of the codebase inside the docker container via `php artisan tinker`, and reports the real output. Use when: validating an Action, DTO, Model, scope or validation rule without writing a test; checking what a method returns for a given input; reproducing a bug with real data; confirming a query result; tinker check; run tinker; execute this in tinker; validate via tinker; run the code and show the output. Not for: writing Pest tests, debugging HTTP requests, or changing database state.'
argument-hint: 'Class + input + what to check (e.g., "What the action class or method returns given the input")'
---

# PHP Tinker Validation

## What It Produces

A verified answer about how the selected code behaves at runtime: the command that was run, the raw output, the exit status, and a short reading of the result against the expected behavior.

## When to Use

- Checking the return value of an Action, DTO, scope, accessor or validation rule with real input
- Reproducing a reported bug against the real application context
- Confirming a rule rejects or accepts a boundary value
- Verifying that a query, relation or resource renders as expected

## Key Terms

| Term | Meaning |
|---|---|
| FQCN | Fully Qualified Class Name — the class name prefixed with its complete namespace path, from the root namespace, so it resolves without a `use` statement. Example: `GustavoMorais\CreateProductAction`. |

## Reach Command

Every run goes through `docker exec`. The command is built from a fixed template and three values, all of them asked from the user on every run. Write it as a single plain line, with no quotes surrounding the whole command.

Template:

```
docker exec --workdir {workdir} {container} {artisan command}
```

Parameters to ask for:

| Parameter | Description |
|---|---|
| `{workdir}` | Absolute path of the project root inside the container |
| `{container}` | Container name or compose service name |
| `{artisan command}` | The `php artisan ...` command to run, without surrounding quotes |

Example:

```
docker exec --workdir /var/www/html/api apicontainer php artisan tinker --execute="echo now()->format('Y-m-d')"
```

## Required Inputs

Ask with the `vscode_askQuestions` tool and stop until complete. Every input is asked on every run; none of them is hardcoded in this skill.

| Input | Description |
|---|---|
| Workdir | Value for `{workdir}` in the reach command template |
| Container | Value for `{container}` in the reach command template |
| Target | Absolute file path and FQCN of the class or method under test |
| Payload | JSON pasted in chat, the exact input the code will receive |
| Expected | The behavior to confirm (return value, exception, count, shape) |
| User ID | The authenticated user ID to bind, when the code resolves the current user |
| Connection | The tenant/DB connection name to use, when the code is tenant-scoped |
| Extra context | Locale, feature flags or config overrides the code depends on |

## Preconditions

- The container is running and reachable from the host terminal.
- The dependency (`laravel/tinker`) is installed in the project under test.
- The workspace agent rule "never run terminal commands" is explicitly lifted for this skill only.

## Procedure

1. Confirm the inputs above with the user before running anything.
2. Read the target file and its direct collaborators to build a correct invocation payload.
3. Bind the context the code needs: authenticated user ID and tenant connection, when applicable.
4. Assemble the command from the [reach command](#reach-command) template, using `php artisan tinker --execute="..."` as the artisan command.
5. Wrap any code path that can persist in `DB::beginTransaction()` and always end with `DB::rollBack()`.
6. Run it in a single sync terminal call and capture stdout, stderr and the exit code.
7. Report the raw output, then state whether it matches the expected behavior.
8. Stop and ask before the next step.

## Safety Rules

- Always read-only: every run that can persist is wrapped in a transaction and ends with a rollback.
- Never pass secrets, credentials or tokens through the questions tool.
- Never edit files to make a run succeed without telling the user.
- Do not chain a run into a fix; the run and the fix are separate steps.

## Output Contract

- The exact command executed
- The raw output, unedited
- The exit status
- The finding, stated as a fact from the output, not an assumption
- The files read during preparation
