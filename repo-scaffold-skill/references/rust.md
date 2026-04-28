# Rust Reference

## Recommended defaults

| Layer           | Tool                                          | Notes                                                |
| --------------- | --------------------------------------------- | ---------------------------------------------------- |
| Package manager | Cargo                                         | Built-in, no alternative                             |
| Task runner     | Taskfile                                      | cargo-make is an alternative but Taskfile is cleaner |
| Linter          | Clippy (cargo clippy)                         | Built-in, excellent                                  |
| Formatter       | rustfmt (cargo fmt)                           | Built-in                                             |
| Type checking   | Built into compiler                           | cargo check is faster than full build                |
| Tests           | cargo test (built-in)                         | No external framework needed for most cases          |
| Commit hooks    | pre-commit or lefthook                        | lefthook is simpler                                  |
| Commit format   | Conventional commits via commitlint or script |                                                      |

## Source directory convention

`src/` — standard Cargo layout. Use `src/` in the docs:check grep pattern.

## Cargo.toml (minimal)

```toml
[package]
name = "your-project"
version = "0.1.0"
edition = "2021"

[dependencies]

[dev-dependencies]

[profile.release]
opt-level = 3
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
    desc: Run clippy and fmt check
    cmds:
      - cargo clippy -- -D warnings
      - cargo fmt --check

  lint:fix:
    desc: Auto-fix formatting
    cmds:
      - cargo fmt

  typecheck:
    desc: Fast type/borrow check without building
    cmds:
      - cargo check

  test:
    desc: Run test suite
    cmds:
      - cargo test

  test:watch:
    desc: Watch mode (requires cargo-watch)
    cmds:
      - cargo watch -x test

  build:
    desc: Build release binary
    cmds:
      - cargo build --release

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

## rustfmt.toml (optional tuning)

```toml
edition = "2021"
max_width = 100
use_small_heuristics = "Default"
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
target/
Cargo.lock    # keep for binaries, add to gitignore for libraries
.env
.DS_Store
```

## Setup commands

```bash
cargo build                  # verify project compiles
cargo install cargo-watch    # optional, for test:watch
cargo install lefthook       # or: use pre-commit
lefthook install
task check
git add -A
task commit MSG="chore: initial scaffold"
```

## Code conventions for CLAUDE.md

- Rust 2021 edition
- Clippy at warning-as-error level in CI
- Prefer `thiserror` for library errors, `anyhow` for application errors
- Tests: unit tests in the same file (`#[cfg(test)]`), integration tests in `tests/`
- No `unwrap()` in library code — use proper error propagation
- Document all public items with `///` doc comments
