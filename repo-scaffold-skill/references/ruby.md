# Ruby Reference

## Recommended defaults

| Layer           | Tool                            | Notes                                                      |
| --------------- | ------------------------------- | ---------------------------------------------------------- |
| Package manager | Bundler (Gemfile)               | Standard                                                   |
| Version manager | rbenv or mise                   | mise is the modern multi-language option                   |
| Task runner     | Taskfile                        | Rakefile is the Ruby-native option but Taskfile is cleaner |
| Linter          | RuboCop                         | Standard; rubocop-rails for Rails projects                 |
| Formatter       | RuboCop (--autocorrect)         | Doubles as formatter                                       |
| Type checking   | Sorbet or Steep + RBS           | Optional but valuable for larger projects                  |
| Tests           | RSpec                           | Standard; minitest is lighter                              |
| Commit hooks    | overcommit or lefthook          | lefthook is simpler                                        |
| Commit format   | Conventional commits via script |                                                            |

## Source directory convention

`lib/` for gems/libraries, `app/` for Rails. Use `lib/` or `app/` in the
docs:check grep pattern depending on project type.

## Gemfile (minimal)

```ruby
source "https://rubygems.org"

ruby "3.3.0"

group :development, :test do
  gem "rspec"
  gem "rubocop"
  gem "rubocop-rspec"
  gem "lefthook"
end
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
    desc: Run RuboCop
    cmds:
      - bundle exec rubocop

  lint:fix:
    desc: Auto-correct RuboCop offenses
    cmds:
      - bundle exec rubocop --autocorrect

  test:
    desc: Run RSpec
    cmds:
      - bundle exec rspec

  test:watch:
    desc: Watch mode (requires guard-rspec)
    cmds:
      - bundle exec guard

  docs:check:
    desc: Verify docs updated when source staged
    cmds:
      - |
        STAGED_SRC=$(git diff --cached --name-only | grep -E '^(lib|app)/' || true)
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

## .rubocop.yml (minimal)

```yaml
AllCops:
  NewCops: enable
  TargetRubyVersion: 3.3
  Exclude:
    - "vendor/**/*"
    - "db/schema.rb"

Layout/LineLength:
  Max: 100

Style/Documentation:
  Enabled: false
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
.bundle/
vendor/bundle/
*.gem
.env
log/
tmp/
coverage/
.DS_Store
```

## Setup commands

```bash
bundle install
bundle exec lefthook install
task check
git add -A
task commit MSG="chore: initial scaffold"
```

## Code conventions for CLAUDE.md

- Ruby 3.3+, frozen string literals enabled
- RuboCop enforces style — don't argue with it, just run `task lint:fix`
- Prefer keyword arguments for methods with more than 2 parameters
- Tests in `spec/`, mirroring source structure
- Avoid meta-programming unless clearly justified
