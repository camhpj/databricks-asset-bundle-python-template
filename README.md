# databricks-asset-bundle-python-template

> Template for a Python package (wheel + entrypoints) that can be deployed as Databricks Jobs via Databricks Asset Bundles (DAB).

## Overview

This repo is meant to be cloned and customized into a Python package that:
- runs locally as a normal Python project; and
- deploys to Databricks as a Job that executes your package entrypoints from a wheel.

Key choices baked in:
- **Databricks Asset Bundles** (`databricks.yml`) to deploy Jobs and related resources.
- **Wheel-first packaging**: Databricks Jobs run `python_wheel_task` against the wheel built from this repo.
- **Python pinned to `==3.12.3`** (configured for Databricks Runtime 17.3 LTS and Databricks Serverless Environment Version 4; adjust as needed).
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
- Databricks CLI with Asset Bundles support

## Install `uv` (Homebrew)
```bash
brew install uv
```

## Install `task` / go-task (Homebrew)
```bash
brew install go-task
```

## Install Databricks CLI (Homebrew)
```bash
brew tap databricks/tap
brew install databricks
```

## Quickstart (development)

```bash
# Create the local virtualenv from uv.lock
task install:frozen

# Run the “local CI” suite (format check, lint, typecheck, tests)
task ci

# Validate and deploy Databricks asset bundle
task bundle:validate
task bundle:deploy
```

## Common commands

All commands below are defined in `Taskfile.yml` and run using the python virtual environment managed by `uv`.

```bash
task fmt              # ruff format
task fmt:check        # ruff format --check
task lint             # ruff check
task lint:fix         # ruff check --fix
task typecheck        # ty check
task test             # pytest
task ci               # fmt:check + lint + typecheck + test
task bundle:validate  # databricks bundle validate
task bundle:deploy    # databricks bundle deploy
```

## Databricks Asset Bundle (DAB)

This template includes a Databricks Asset Bundle in `databricks.yml` and two example Jobs under `resources/`:
- `resources/sample.job.yml`: runs on a job cluster and installs `dist/*.whl` as a library.
- `resources/sample_serverless.job.yml`: runs on Serverless and installs `dist/*.whl` via an environment dependency.

Helpful docs:
- Databricks Asset Bundles: https://docs.databricks.com/dev-tools/bundles/index.html
- `python_wheel_task` (Jobs): https://docs.databricks.com/en/jobs/how-to/use-python-wheels.html

Both examples use a `python_wheel_task` that runs:
- `package_name: python_template`
- `entry_point: main` (configured in `pyproject.toml` under `[project.scripts]`)

### Configure your workspace and identity

1) Update the Databricks workspace host and production deployment settings in `databricks.yml`.
   - Replace `https://company.databricks.com` with your workspace URL.
   - In `targets.prod.workspace.root_path`, replace the placeholder user path with your desired deployment path.
   - Update `targets.prod.permissions` to match your org (or remove the block if you manage permissions elsewhere).

2) Authenticate the Databricks CLI (profile or environment variables).
   - Prefer a service principal where possible. Developers can add a service principal profile to `~/.databrickscfg`:
     - This uses OAuth client credentials; the CLI will obtain OAuth access tokens on your behalf.

     ```ini
     [service-principal-name]
     host          = https://{host}.cloud.databricks.com
     client_id     = {client_id}
     client_secret = {client_secret}
     ```

   - Then set `DATABRICKS_CONFIG_PROFILE=service-principal-name` when running `databricks bundle ...` commands (or use a shell profile/direnv to set it automatically).
   - For interactive development, you can also use a CLI auth/profile workflow (check `databricks auth --help`) or set `DATABRICKS_HOST` + `DATABRICKS_TOKEN` (PAT).

### Validate, deploy, and run

The bundle is configured to build a wheel artifact with:
- `uv build --wheel`

Common workflows:

```bash
# Ensure the CLI is using the right profile (prefer service principals for deploys)
export DATABRICKS_CONFIG_PROFILE=service-principal-name

# Validate the bundle configuration
TARGET=dev task bundle:validate
# or: databricks bundle validate -t dev

# Deploy to the default target (dev)
TARGET=dev task bundle:deploy
# or: databricks bundle deploy -t dev

# Run a job defined in resources/
databricks bundle run sample_job
# or:
databricks bundle run sample_serverless_job

# Deploy / run in prod explicitly
TARGET=prod task bundle:deploy
databricks bundle run -t prod sample_job

# Tear down deployed resources for a target
databricks bundle destroy -t dev
```

Notes:
- The example cluster job uses Databricks Runtime 17.3 LTS (`spark_version: 17.3.x-scala2.13`). Update this (and node types / cloud attributes) to match your workspace.
- The example serverless job uses `environment_version: "4"`. Update if your workspace requires a different serverless environment version.

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
├─ databricks.yml               # bundle definition (targets, artifacts, variables)
├─ resources/                   # bundle resources (jobs, pipelines, etc.)
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
- Update Databricks bundle/job references:
  - `databricks.yml: bundle.name`
  - `resources/*.yml: python_wheel_task.package_name`
  - `pyproject.toml: [project.scripts]` entrypoints (and update `resources/*.yml: python_wheel_task.entry_point` to match)
- Update tool config references:
  - `tool.ruff.lint.isort.known-first-party`
  - `tool.coverage.run.source`
