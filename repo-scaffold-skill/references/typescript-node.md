# TypeScript / Node.js Reference

## Recommended defaults

| Layer           | Tool                                         | Notes                                          |
| --------------- | -------------------------------------------- | ---------------------------------------------- |
| Package manager | pnpm                                         | Faster, stricter isolation than npm/yarn       |
| Task runner     | Taskfile                                     | Language-agnostic, clean YAML                  |
| Linter          | ESLint (flat config)                         | eslint.config.js, v9+                          |
| Formatter       | Prettier                                     | .prettierrc                                    |
| Tests           | Vitest                                       | Fast, ESM-native; Jest is fine if preferred    |
| Commit hooks    | Husky v9                                     | pnpm-compatible, runs via package.json prepare |
| Commit format   | commitlint + @commitlint/config-conventional |                                                |
| Type checking   | tsc --noEmit                                 | Add to check task                              |

## Source directory convention

`src/` — use this in the docs:check grep pattern.

## package.json scripts

```json
{
  "scripts": {
    "prepare": "husky",
    "test": "vitest run",
    "lint": "eslint . --max-warnings 0",
    "lint:fix": "eslint . --fix && prettier --write .",
    "format:check": "prettier --check .",
    "typecheck": "tsc --noEmit"
  },
  "devDependencies": {
    "@commitlint/cli": "^19.0.0",
    "@commitlint/config-conventional": "^19.0.0",
    "eslint": "^9.0.0",
    "husky": "^9.0.0",
    "prettier": "^3.0.0",
    "typescript": "^5.0.0",
    "vitest": "^2.0.0"
  },
  "packageManager": "pnpm@9.0.0",
  "engines": {
    "node": ">=20.0.0",
    "pnpm": ">=9.0.0"
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
    desc: Run ESLint and Prettier check
    cmds:
      - pnpm eslint . --max-warnings 0
      - pnpm prettier --check .

  lint:fix:
    desc: Auto-fix lint and formatting
    cmds:
      - pnpm eslint . --fix
      - pnpm prettier --write .

  typecheck:
    desc: TypeScript type check
    cmds:
      - pnpm tsc --noEmit

  test:
    desc: Run test suite
    cmds:
      - pnpm vitest run

  test:watch:
    desc: Run tests in watch mode
    cmds:
      - pnpm vitest

  docs:check:
    desc: Verify docs updated when source staged
    cmds:
      - |
        STAGED_SRC=$(git diff --cached --name-only | grep -E '^src/' || true)
        STAGED_DOCS=$(git diff --cached --name-only | grep -E '^docs/' || true)
        if [ -n "$STAGED_SRC" ] && [ -z "$STAGED_DOCS" ]; then
          echo ""; echo "  ✗ DOCS CHECK FAILED"
          echo "  Source files are staged but no docs/ files were updated."
          echo "  Update docs/plans/, docs/adr/, docs/specs/, or docs/chats/"
          echo "  then stage the file and run 'task commit' again."; echo ""
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
        echo "# {{.NEXT}}. [Title]\n\nDate: $(date +%Y-%m-%d)\nStatus: Proposed\n\n## Context\n\n## Decision\n\n## Consequences\n\n## Alternatives Considered\n" > "$FILE"
        echo "Created $FILE"

  spec:
    desc: "Scaffold a new feature spec (set NAME=feature-name)"
    vars:
      NAME: '{{.NAME | default "new-feature"}}'
    cmds:
      - |
        FILE="docs/specs/{{.NAME}}.md"
        echo "# Feature Spec: {{.NAME}}\n\nDate: $(date +%Y-%m-%d)\nStatus: Draft\n\n## Overview\n\n## Scope\n\n### In scope\n\n### Out of scope\n\n## Acceptance Criteria\n\n- [ ]\n\n## Open Questions\n\n- [ ]\n" > "$FILE"
        echo "Created $FILE"

  summary:
    desc: Print the chat summary prompt to paste into Claude
    cmds:
      - echo "\n  Paste into Claude:\n  Write a chat summary to docs/chats/$(date +%Y-%m-%d)-<topic>.md\n  covering: what we tried to do, decisions made, what was deferred, follow-up tasks.\n"

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

## Husky hooks

`.husky/pre-commit`:

```sh
#!/bin/sh
task docs:check
```

`.husky/commit-msg`:

```sh
#!/bin/sh
pnpm commitlint --edit "$1"
```

## commitlint.config.js

```js
export default {
  extends: ["@commitlint/config-conventional"],
  rules: {
    "header-max-length": [2, "always", 100],
    "type-enum": [
      2,
      "always",
      [
        "feat",
        "fix",
        "docs",
        "style",
        "refactor",
        "test",
        "chore",
        "ci",
        "perf",
        "revert",
      ],
    ],
  },
};
```

## eslint.config.js

```js
export default [
  { ignores: ["node_modules/**", "dist/**", "coverage/**"] },
  {
    rules: {
      "no-unused-vars": "warn",
      "no-console": "warn",
      "prefer-const": "error",
    },
  },
];
```

## .prettierrc

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

## tsconfig.json (strict baseline)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## .gitignore

```
node_modules/
dist/
build/
.next/
coverage/
.env
.env.local
*.log
.DS_Store
```

## Setup commands

```bash
pnpm install        # installs deps and runs husky prepare
task check          # verify everything works
git add -A
task commit MSG="chore: initial scaffold"
```

## Code conventions for CLAUDE.md

- Language: TypeScript strict mode
- Imports: absolute from src/ root preferred
- Test files: colocate as `*.test.ts`
- No `any` without a comment explaining why
- Prefer named exports over default exports
