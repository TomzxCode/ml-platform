# Conventions

Defaults: Python community standards plus the tooling defaults already chosen.

## Naming

| Item | Rule |
|---|---|
| Files and modules | snake_case |
| Functions and variables | snake_case |
| Classes | PascalCase |
| Constants | UPPER_SNAKE_CASE |
| CLI commands | kebab-case (cyclopts default) |

## Directory Structure

- src layout: `src/<package>/`.
- Separate packages for CLI and server within one repository.

## Coding Standards

- PEP 8, enforced by ruff defaults.
- Type hints required on all public functions.
- Keep `__init__.py` files minimal/empty, only for package initialization.
- No `__all__` in `__init__.py` files.

## Commits and Branching

- Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`, ...).
- Feature branches named `feat/<slug>`, `fix/<slug>`.
- Changes land via pull request.

## Document Style

- Markdown, one sentence per line.
- Tables for structured comparisons.
- No em-dashes; use commas or parentheses.
