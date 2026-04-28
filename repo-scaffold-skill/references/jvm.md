# JVM Reference (Java / Kotlin)

## Recommended defaults

| Layer                   | Tool                                 | Notes                                        |
| ----------------------- | ------------------------------------ | -------------------------------------------- |
| Build / package manager | Gradle (Kotlin DSL)                  | Modern default; Maven if team prefers        |
| Task runner             | Taskfile                             | Wraps Gradle tasks with consistent interface |
| Linter                  | Detekt (Kotlin) / Checkstyle (Java)  | Detekt is standard for Kotlin                |
| Formatter               | ktlint (Kotlin) / google-java-format |                                              |
| Type checking           | Built into compiler                  |                                              |
| Tests                   | JUnit 5 + AssertJ                    | Kotest is a good alternative for Kotlin      |
| Commit hooks            | lefthook                             | Simple, fast                                 |
| Commit format           | Conventional commits via script      |                                              |

## Source directory convention

`src/main/kotlin/` or `src/main/java/`. Use `src/` in the docs:check pattern.

## build.gradle.kts (Kotlin, minimal)

```kotlin
plugins {
  kotlin("jvm") version "2.0.0"
  id("io.gitlab.arturbosch.detekt") version "1.23.0"
  id("org.jlleitschuh.gradle.ktlint") version "12.0.0"
  application
}

group = "com.yourcompany"
version = "0.1.0"

repositories {
  mavenCentral()
}

dependencies {
  testImplementation(kotlin("test"))
  testImplementation("org.assertj:assertj-core:3.25.0")
  testImplementation("org.junit.jupiter:junit-jupiter:5.10.0")
}

tasks.test {
  useJUnitPlatform()
}

kotlin {
  jvmToolchain(21)
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
      - task: test
      - task: docs:check

  lint:
    desc: Run detekt and ktlint check
    cmds:
      - ./gradlew detekt ktlintCheck

  lint:fix:
    desc: Auto-format with ktlint
    cmds:
      - ./gradlew ktlintFormat

  test:
    desc: Run tests
    cmds:
      - ./gradlew test

  build:
    desc: Build the project
    cmds:
      - ./gradlew build

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
.gradle/
build/
*.class
*.jar
.idea/
*.iml
.env
.DS_Store
```

## Setup commands

```bash
./gradlew wrapper          # ensure gradle wrapper present
go install github.com/evilmartians/lefthook@latest
lefthook install
task check
git add -A
task commit MSG="chore: initial scaffold"
```

## Code conventions for CLAUDE.md

- Java 21+ / Kotlin 2.0+
- Kotlin preferred for new code; Java interop is fine
- Immutability by default: `val` over `var`, data classes for value types
- Tests colocated in `src/test/kotlin/` mirroring the main source tree
- Coroutines for async work (Kotlin); virtual threads (Java 21+) acceptable
