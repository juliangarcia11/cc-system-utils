# Python Reference

## Recommended defaults

| Layer           | Tool                | Notes                                          |
| --------------- | ------------------- | ---------------------------------------------- |
| Package manager | uv                  | Modern, fast; pip + venv is the fallback       |
| Task runner     | Taskfile            | Language-agnostic                              |
| Linter          | Ruff                | Replaces flake8, isort, pyupgrade — all in one |
| Formatter       | Ruff format         | Replaces Black                                 |
| Type checking   | mypy or pyright     | mypy is more established; pyright is faster    |
| Tests           | pytest + pytest-cov | Standard; add pytest-watch for watch mode      |
| Commit hooks    | pre-commit          | Python-native, widely used                     |
| Commit format   | commitizen (cz)     | Python-native conventional commits             |

## Source directory convention

`src/<package_name>/` (src layout) — use `src/` in the docs:check grep pattern.

## pyproject.toml (complete)

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "your-project"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = []

[project.optional-dependencies]
dev = [
  "ruff>=0.6.0",
  "mypy>=1.0.0",
  "pytest>=8.0.0",
  "pytest-cov>=5.0.0",
  "commitizen>=3.0.0",
  "pre-commit>=3.0.0",
]

[tool.ruff]
line-length = 100
src = ["src"]

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "N", "B"]
ignore = []

[tool.ruff.format]
quote-style = "double"

[tool.mypy]
strict = true
python_version = "3.12"

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--cov=src --cov-report=term-missing --cov-fail-under=80"

[tool.commitizen]
name = "cz_conventional_commits"
version = "0.1.0"
tag_format = "v$version"
```

## Taskfile.yml (complete)

```yaml
version: "3"

tasks:
  check:
    desc: Run all checks
    cmds:
      - task: lint
      - task: typecheck
      - task: test
      - task: docs:check

  lint:
    desc: Ruff lint and format check
    cmds:
      - uv run ruff check .
      - uv run ruff format --check .

  lint:fix:
    desc: Auto-fix lint and formatting
    cmds:
      - uv run ruff check --fix .
      - uv run ruff format .

  typecheck:
    desc: Run mypy type checking
    cmds:
      - uv run mypy src/

  test:
    desc: Run pytest
    cmds:
      - uv run pytest

  test:watch:
    desc: Run pytest in watch mode
    cmds:
      - uv run pytest-watch

  docs:check:
    desc: Verify docs updated when source staged
    cmds:
      - |
        STAGED_SRC=$(git diff --cached --name-only | grep -E '^src/' || true)
        STAGED_DOCS=$(git diff --cached --name-only | grep -E '^docs/' || true)
        if [ -n "$STAGED_SRC" ] && [ -z "$STAGED_DOCS" ]; then
          echo ""; echo "  ✗ DOCS CHECK FAILED"
          echo "  Source files staged but no docs/ files updated."
          echo "  Update docs/plans/, docs/adr/, docs/specs/, or docs/chats/"
          echo "  then stage and run 'task commit' again."; echo ""
          exit 1
        fi
        echo "  ✓ Docs check passed"

  adr:
    desc: "Scaffold a new ADR (set SLUG=short-title)"
    vars:
      NEXT:
        sh: printf "%04d" $(($(ls docs/adr/*.md 2>/dev/null | grep -oE '[0-9]{4}' | sort | tail -1 || echo 0) + 1))
      SLUG: '{{.SLUG | default "new-decision"}}'
    cmds:
      - |
        FILE="docs/adr/{{.NEXT}}-{{.SLUG}}.md"
        printf "# {{.NEXT}}. [Title]\n\nDate: $(date +%%Y-%%m-%%d)\nStatus: Proposed\n\n## Context\n\n## Decision\n\n## Consequences\n\n## Alternatives Considered\n" > "$FILE"
        echo "Created $FILE"

  spec:
    desc: "Scaffold a new spec (set NAME=feature-name)"
    vars:
      NAME: '{{.NAME | default "new-feature"}}'
    cmds:
      - |
        FILE="docs/specs/{{.NAME}}.md"
        printf "# Feature Spec: {{.NAME}}\n\nDate: $(date +%%Y-%%m-%%d)\nStatus: Draft\n\n## Overview\n\n## Acceptance Criteria\n\n- [ ]\n\n## Open Questions\n\n- [ ]\n" > "$FILE"
        echo "Created $FILE"

  summary:
    desc: Print chat summary prompt
    cmds:
      - echo "\n  Paste into Claude:\n  Write a chat summary to docs/chats/$(date +%Y-%m-%d)-<topic>.md\n"

  commit:
    desc: "Run checks then commit (set MSG='type(scope): description')"
    cmds:
      - task: check
      - |
        if [ -z "{{.MSG}}" ]; then
          echo "Usage: task commit MSG='feat(scope): description'"; exit 1
        fi
      - git commit -m "{{.MSG}}"

  commit:quick:
    desc: Stage all and commit
    cmds:
      - git add -A
      - task: commit
        vars: { MSG: "{{.MSG}}" }
```

## .pre-commit-config.yaml

```yaml
repos:
  - repo: local
    hooks:
      - id: docs-check
        name: Docs check
        entry: task docs:check
        language: system
        pass_filenames: false
        always_run: false
        stages: [pre-commit]
      - id: commitizen
        name: Conventional commit format
        entry: cz check --commit-msg-file
        language: system
        stages: [commit-msg]
```

## .gitignore

```
__pycache__/
*.py[cod]
*.egg-info/
.venv/
dist/
build/
.mypy_cache/
.ruff_cache/
.pytest_cache/
htmlcov/
.coverage
.env
.DS_Store
```

## Setup commands

```bash
uv sync --extra dev          # install all dependencies
pre-commit install           # install git hooks
pre-commit install --hook-type commit-msg
task check                   # verify everything works
git add -A
task commit MSG="chore: initial scaffold"
```

## Code conventions for CLAUDE.md

- Python 3.12+, type annotations required on all public functions
- src layout: all code under `src/<package>/`
- Tests in `tests/`, mirroring the src structure
- Imports: stdlib → third-party → local, sorted by Ruff
- Prefer dataclasses or Pydantic models over plain dicts for structured data
