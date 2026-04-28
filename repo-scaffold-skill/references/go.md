# Go Reference

## Recommended defaults

| Layer           | Tool                       | Notes                                          |
| --------------- | -------------------------- | ---------------------------------------------- |
| Package manager | Go modules (go mod)        | Built-in, no alternative needed                |
| Task runner     | Taskfile                   | Go-compatible, used by Go community            |
| Linter          | golangci-lint              | Aggregates dozens of linters                   |
| Formatter       | gofmt / goimports          | Built-in; goimports adds import management     |
| Type checking   | Built into compiler        | No separate tool needed                        |
| Tests           | testing (stdlib) + testify | testify adds assertions; go test is the runner |
| Commit hooks    | pre-commit or lefthook     | lefthook is Go-native and fast                 |
| Commit format   | commitlint or manual       | conventional-commit-go is a lighter option     |

## Source directory convention

Go source is at the module root or in `cmd/` and `internal/`. Use the module
root for the docs:check grep pattern — check for `.go` files rather than a
fixed directory prefix.

## go.mod (minimal)

```
module github.com/yourname/your-project

go 1.23
```

## Taskfile.yml (complete)

```yaml
version: "3"

tasks:
  check:
    desc: Run all checks
    cmds:
      - task: lint
      - task: test
      - task: docs:check

  lint:
    desc: Run golangci-lint
    cmds:
      - golangci-lint run ./...

  lint:fix:
    desc: Run goimports and gofmt
    cmds:
      - goimports -w .
      - gofmt -w .

  test:
    desc: Run tests with coverage
    cmds:
      - go test -race -coverprofile=coverage.out ./...
      - go tool cover -func=coverage.out

  test:watch:
    desc: Watch and rerun tests (requires gotestsum)
    cmds:
      - gotestsum --watch ./...

  build:
    desc: Build the binary
    cmds:
      - go build -o bin/app ./cmd/...

  docs:check:
    desc: Verify docs updated when Go source staged
    cmds:
      - |
        STAGED_SRC=$(git diff --cached --name-only | grep -E '\.go$' || true)
        STAGED_DOCS=$(git diff --cached --name-only | grep -E '^docs/' || true)
        if [ -n "$STAGED_SRC" ] && [ -z "$STAGED_DOCS" ]; then
          echo ""; echo "  ✗ DOCS CHECK FAILED"
          echo "  Go source files staged but no docs/ files updated."
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

## .golangci-lint.yml (minimal)

```yaml
linters:
  enable:
    - gofmt
    - goimports
    - govet
    - staticcheck
    - errcheck
    - gosimple
    - unused

linters-settings:
  goimports:
    local-prefixes: github.com/yourname/your-project
```

## lefthook.yml (commit hooks)

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
          echo "✗ Commit message must follow Conventional Commits format"
          echo "  Example: feat(auth): add login handler"
          exit 1
        fi
```

## .gitignore

```
bin/
*.exe
*.test
*.out
coverage.out
.env
.DS_Store
```

## Setup commands

```bash
go mod tidy                  # initialize/tidy modules
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install github.com/evilmartians/lefthook@latest
lefthook install             # install git hooks
task check                   # verify everything works
git add -A
task commit MSG="chore: initial scaffold"
```

## Code conventions for CLAUDE.md

- Go 1.23+, idiomatic Go style (gofmt is law)
- Package structure: `cmd/` for binaries, `internal/` for private packages, `pkg/` for public
- Error handling: always check errors, never ignore with `_` unless intentional
- Tests: table-driven tests preferred, `*_test.go` colocated with source
- Interfaces: define at point of use (consumer), not at point of implementation
