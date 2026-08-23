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
- Server logging uses structlog, with a request identifier on every request.

## CLI Conventions

Fixed by the FEAT-p1 plan; refined per command group by FEAT-p3 to FEAT-p14 and mirrored in the user-facing docs (`docs/guides/cli-conventions.md`).

- Binary name: `mlx`.
- Command groups are nouns with verb-first subcommands (`mlx dataset list`).
- Command and option names are kebab-case; user-visible resource names (workspaces, datasets, secrets, experiments, models) are kebab-case.
- Global options: `--profile <name>`, `--workspace <name>` (per-invocation override of the persisted selection), `--output <format>`, `--json`, `--log-level <level>`, `--verbose`, `--version`, `--help`.
- `--json` is shorthand for `--output json`; `--verbose` is shorthand for `--log-level debug`.
- Output formats for list and get commands: `json`, `yaml`, `table` (default), `name` (one element name per line, for shell piping).
- Log levels: `debug`, `info`, `warn` (default), `error`; logging goes to stderr, never stdout.
- Exit codes: 0 success, 1 usage error, 2 authentication failure, 3 server error, 4 cancelled by user.
- stdout carries results only; prompts, progress, and diagnostics go to stderr.
- `--output json` prints exactly one valid JSON document to stdout and nothing else.
- Errors print `error: <message>` followed by `hint: <fix>`.
- Tokens and secret values are never echoed, logged, or printed.

## API Conventions

Fixed by the FEAT-p2 contract plan.

- Workspace-scoped resources live under `/workspaces/{workspace}/...`.
- List endpoints paginate with cursors.
- Submission endpoints accept an `Idempotency-Key` header with replay semantics.
- Every failure returns the shared error model: code, message, likely cause, suggested fix.
- Authentication is a bearer token; tokens are stored hashed at rest and never logged.

## Input Files

- Job, dataset, experiment, and cluster specs are YAML files.
- Model references use `name:version`: kebab-case name plus a positive integer version (the v1 scheme; defaults to the next version under the name when omitted).

## Telemetry Conventions

Established by the telemetry plans of FEAT-p3, p8, p9, p10, p12, and p13.

- Events are snake_case, named `<group>_<command>_<outcome>` (for example `job_submit_succeeded`, `auth_login_failed`).
- All events carry shared properties: `source: "cli"`, the anonymous `install_id`, and `cli_version`.
- No event carries the token, user identity, server URL, or profile name.
- Telemetry is anonymous and opt-out.

## Documentation

- User documentation is a MkDocs Material site in `docs/` (`index`, `installation`, `quickstart`, `concepts/`, `guides/`, `reference/cli/`, `reference/specs/`).
- Serve locally with `uv run mkdocs serve` (the `docs` dependency group).
- CLI reference pages stay in sync with the FEAT-p1 command reference; spec reference pages mirror the YAML spec schemas.

## Commits and Branching

- Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`, ...).
- Feature branches named `feat/<slug>`, `fix/<slug>`.
- Changes land via pull request.

## Document Style

- Markdown, one sentence per line.
- Tables for structured comparisons.
- No em-dashes; use commas or parentheses.
