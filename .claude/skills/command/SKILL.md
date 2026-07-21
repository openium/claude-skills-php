---
name: command
description: "Generates a Symfony console command. Produces a command with arguments, options, input validation, progress bar, structured output, and error handling."
---

# Symfony Console Command Generation

## Structure

### PHP 8+ Attribute

```php
#[AsCommand(
    name: 'app:import-csv',
    description: 'Short description',
    aliases: ['a:ic'],
)]
```

### Naming

- Prefix `app:`, format `app:action-name`.
- Use kebab-case for compound segments (`import-users`, `sync-products`, `delete-expired-sessions`).
- Keep segments short, explicit, and in English.

### Description

- Use a short and clear English sentence that describes the action.

### Alias

- Add a short alias in the `a:abc` format.
- `abc` matches the first meaningful letters of the action name.
- Example: `app:import-csv` -> alias `a:ic`.
- Example: `app:sync-prices` -> alias `a:sp`.
- Do not create an ambiguous alias if several commands would have the same alias.

## Rules

### Architecture

- The command is a thin adapter: parse inputs, call a service, display the result.
- No business logic in the command.
- Inject dependencies through the constructor.

### Input

- Define arguments and options in `configure()` with explicit names.
- Type and validate each value retrieved from `InputInterface`.
- Use `InputArgument::REQUIRED` only for indispensable values.
- Use options for behaviors (`--dry-run`, `--force`, `--limit`, `--batch-size`, `--env`).
- Provide a safe default value for numeric options.
- Reject invalid values with a clear message and `Command::INVALID`.
- Ask for confirmation with `$io->confirm()` before a destructive action, unless `--force` is provided.
- Never read `$_SERVER`, `$_ENV`, `$_GET`, or `$_POST` directly in the command.

### Interactive Questions

- Use `SymfonyStyle` helpers (`ask()`, `choice()`, `confirm()`) for simple questions.
- Use `QuestionHelper` and the `Question`, `ChoiceQuestion`, `ConfirmationQuestion` classes for advanced cases.
- Ask a question only if the value is not provided by an argument or option.
- Plan for non-interactive mode: if `--no-interaction` is active, use default values or return `Command::INVALID`.
- Always provide an explicit default value when the command can continue without an answer.
- Validate the answer with a question validator or a dedicated method.
- Hide secrets with `Question::setHidden(true)` and never display them in output.
- For choices, use `ChoiceQuestion` with a closed list and reject answers outside the list.
- For destructive actions, ask for confirmation with `confirm()` or `ConfirmationQuestion`, unless `--force` is provided.

### Output

- Use `SymfonyStyle`.
- `$io->title()`, `$io->table()`, `$io->progressBar()`
- `$io->success()` / `$io->error()` for the final result
- Adapt detail with verbosity levels (`-v`, `-vv`, `-vvv`)
- Display a final summary: number of processed, skipped, failed items
- Use `$io->warning()` for non-blocking cases and `$io->note()` for useful information
- For errors, display an actionable message without exposing secrets or sensitive data
- Return the right code: `Command::SUCCESS`, `Command::FAILURE`, or `Command::INVALID`
- Do not write directly with `echo`, `var_dump()`, or `print_r()`
- Keep output stable and readable for execution in CI or cron

### Dry-run

- Support `--dry-run` when the command modifies data.
- Display what would be done without persisting.

### Batch Processing

Flush and clear Doctrine every 100 items in loops.

## Output Format

Complete command class with imports, attribute, configuration, and execute method.
