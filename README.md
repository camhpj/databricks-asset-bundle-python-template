# python-template

> Template for Python libraries/packages using `uv` + `go-task`, with formatting/linting via `ruff`, tests via `pytest`, and type checks via `ty`.

## Overview

This repo is meant to be cloned and customized into a publishable Python package.

Key choices baked in:
- **Python pinned to `==3.12.3`** (useful for Databricks Runtime compatibility).
- **`uv`** for dependency management and running tools (`uv sync`, `uv run …`).
- **`go-task`** as a thin command runner (see `Taskfile.yml`).
- **`ruff`** for formatting + linting.
- **`pytest`** (+ `pytest-cov`) for tests and coverage.
- **`ty`** for type checking.
- **Commitizen** for Conventional Commits and version bumping.

## Requirements

- Unix-like OS (macOS/Linux)
- Python `==3.12.3`
- `uv`
- `task` (go-task)

## Install `uv` (Unix)
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Install `task` / go-task (Unix)
```bash
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b ~/.local/bin
```

## Quickstart (development)

```bash
# Create the local virtualenv from uv.lock
task install:frozen

# Run the “local CI” suite (format check, lint, typecheck, tests)
task ci
```

## Common commands

All commands below are defined in `Taskfile.yml` and run using the python virtual environment managed by `uv`.

```bash
task fmt          # ruff format
task fmt:check    # ruff format --check
task lint         # ruff check
task lint:fix     # ruff check --fix
task typecheck    # ty check
task test         # pytest
task ci           # fmt:check + lint + typecheck + test
```

If you prefer to bypass `task`, the equivalent pattern is:

```bash
uv run <tool> <args>
```

## Dependency management

This template uses `uv.lock` for reproducible environments.

Typical workflows:
- Add a runtime dependency: `uv add <package>`
- Add a dev dependency group dep: `uv add --group dev <package>`
- Update the lockfile: `task install`
- Sync environment (strict): `task install:frozen`

## Project layout

```text
.
├─ src/
│  └─ python_template/          # rename this package
├─ pyproject.toml               # project metadata + tool config
├─ uv.lock                      # locked dependencies (commit this)
└─ Taskfile.yml                 # task runner entrypoints
```

## Versioning & release (optional)

This template is configured for Commitizen + Conventional Commits.

```bash
# Bump version, update changelog, and create a tag (per commitizen config)
uv run cz bump

# Build distribution artifacts
uv build

# Publish (configure credentials first)
uv publish
```

## Customizing this template

Minimum rename checklist:
- Update `[project]` metadata in `pyproject.toml` (`name`, `description`, `authors`, etc.).
- Rename the import package folder: `src/python_template/` → `src/<your_package>/`.
- Update tool config references:
  - `tool.ruff.lint.isort.known-first-party`
  - `tool.coverage.run.source`
