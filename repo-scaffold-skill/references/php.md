# PHP Reference

## Recommended defaults

| Layer           | Tool                            | Notes                                             |
| --------------- | ------------------------------- | ------------------------------------------------- |
| Package manager | Composer                        | Standard                                          |
| Task runner     | Taskfile                        | language-agnostic; can also wrap composer scripts |
| Linter          | PHP_CodeSniffer or PHP-CS-Fixer | PHP-CS-Fixer is more modern                       |
| Formatter       | PHP-CS-Fixer                    | Doubles as linter                                 |
| Type checking   | PHPStan (level 8+)              | Psalm is a good alternative                       |
| Tests           | Pest                            | Modern, expressive; PHPUnit is the classic        |
| Commit hooks    | lefthook or CaptainHook         | lefthook is simpler                               |
| Commit format   | Conventional commits via script |                                                   |

## Source directory convention

`src/` — PSR-4 standard. Use `src/` in the docs:check grep pattern.

## composer.json (minimal)

```json
{
  "name": "yourname/your-project",
  "require": {
    "php": ">=8.3"
  },
  "require-dev": {
    "pestphp/pest": "^3.0",
    "phpstan/phpstan": "^1.10",
    "friendsofphp/php-cs-fixer": "^3.0"
  },
  "autoload": {
    "psr-4": {
      "App\\": "src/"
    }
  },
  "autoload-dev": {
    "psr-4": {
      "Tests\\": "tests/"
    }
  }
}
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
    desc: Run PHP-CS-Fixer in dry-run mode
    cmds:
      - vendor/bin/php-cs-fixer fix --dry-run --diff

  lint:fix:
    desc: Auto-fix code style
    cmds:
      - vendor/bin/php-cs-fixer fix

  typecheck:
    desc: Run PHPStan
    cmds:
      - vendor/bin/phpstan analyse src --level=8

  test:
    desc: Run Pest tests
    cmds:
      - vendor/bin/pest

  test:watch:
    desc: Watch mode (requires fswatch)
    cmds:
      - fswatch -o src/ tests/ | xargs -n1 -I{} vendor/bin/pest

  docs:check:
    desc: Verify docs updated when source staged
    cmds:
      - |
        STAGED_SRC=$(git diff --cached --name-only | grep -E '^src/' || true)
        STAGED_DOCS=$(git diff --cached --name-only | grep -E '^docs/' || true)
        if [ -n "$STAGED_SRC" ] && [ -z "$STAGED_DOCS" ]; then
          echo ""; echo "  ✗ DOCS CHECK FAILED"
          echo "  Source files staged but no docs/ files updated."
          echo "  Update docs/plans/, docs/adr/, docs/specs/, or docs/chats/"; echo ""
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
        printf "# {{.NEXT}}. [Title]\n\nDate: $(date +%%Y-%%m-%%d)\nStatus: Proposed\n\n## Context\n\n## Decision\n\n## Consequences\n" > "$FILE"
        echo "Created $FILE"

  spec:
    desc: "Scaffold a new spec (set NAME=feature-name)"
    vars:
      NAME: '{{.NAME | default "new-feature"}}'
    cmds:
      - |
        FILE="docs/specs/{{.NAME}}.md"
        printf "# Feature Spec: {{.NAME}}\n\nDate: $(date +%%Y-%%m-%%d)\nStatus: Draft\n\n## Overview\n\n## Acceptance Criteria\n\n- [ ]\n" > "$FILE"
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

## lefthook.yml

```yaml
pre-commit:
  commands:
    docs-check:
      run: task docs:check

commit-msg:
  commands:
    conventional:
      run: |
        MSG=$(cat {1})
        if ! echo "$MSG" | grep -qE '^(feat|fix|docs|style|refactor|test|chore|ci|perf|revert)(\(.+\))?: .+'; then
          echo "✗ Commit message must follow Conventional Commits"
          exit 1
        fi
```

## .gitignore

```
vendor/
.php-cs-fixer.cache
.phpunit.result.cache
.env
.DS_Store
```

## Setup commands

```bash
composer install
go install github.com/evilmartians/lefthook@latest
lefthook install
task check
git add -A
task commit MSG="chore: initial scaffold"
```

## Code conventions for CLAUDE.md

- PHP 8.3+, strict types enabled in all files (`declare(strict_types=1)`)
- PSR-4 autoloading, PSR-12 code style (enforced by PHP-CS-Fixer)
- PHPStan at level 8 — type everything, no `mixed` without justification
- Tests in `tests/`, mirroring `src/` structure
- Prefer named constructors over public `__construct` for domain objects
