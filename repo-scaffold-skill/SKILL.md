---
name: repo-scaffold-skill
description: Interviews the developer about their project stack and goals, then scaffolds
a complete starter repository with CLAUDE.md, documentation folders (docs/adr,
docs/chats, docs/plans, docs/specs), language-appropriate tooling equivalents
for linting, testing, formatting, commit hooks, and task running, plus all
config files ready to use. Use this skill whenever a user wants to bootstrap
a new project, start a new repo, set up a Claude-assisted workflow, scaffold
a starter codebase, or create a project template — even if they don't use the
words "scaffold" or "template". Also trigger when the user mentions wanting
to replicate a workflow across projects, set up docs-as-context, or configure
Claude for a new codebase from scratch.
---

# Repo Scaffold Skill

This skill interviews the developer about their stack and goals, then generates
a complete starter repo tailored to their language and tooling — with the
docs-as-context workflow baked in from the start.

## Overview of what gets built

Every scaffold produces:

- `CLAUDE.md` — workflow rules, conventions, task reference, customization checklist
- `docs/adr/` — Architecture Decision Records (template + first real ADR)
- `docs/chats/` — chat summary from the bootstrapping session
- `docs/plans/` — plans folder with usage README
- `docs/specs/` — specs folder with usage README
- A task runner config (`Taskfile.yml` or equivalent)
- A commit hook setup (Husky, pre-commit, lefthook, or language equivalent)
- Lint, format, and test tool configs
- `README.md` explaining the repo

---

## Phase 1: Interview

Conduct the interview **conversationally** — don't dump all questions at once.
Start with the most important question, branch based on the answer, and only ask
what you don't already know from context.

### Core questions (ask in order, skip if already known)

1. **Language / runtime** — "What language and runtime are you building with?"
   This is the branching decision. Everything else follows from it.

2. **Project type** — What kind of thing is this? (web app, API, CLI, library,
   data pipeline, mobile, etc.) This shapes which tools make sense.

3. **Existing preferences** — Do they have strong opinions on specific tools
   (test framework, formatter, task runner)? If yes, use those. If no, suggest
   the recommended defaults from the language reference.

4. **Team size and CI** — Solo or team? CI/CD in scope now or later?
   (Affects whether to set up commitlint strictly, add CI config, etc.)

5. **Special requirements** — Monorepo? Docker? Specific database or cloud?
   These add to the scaffold but don't change the core structure.

After the first answer, you have enough to load the right language reference.
Read it before asking follow-up questions — it will tell you the right defaults
to suggest.

### Loading language references

Based on the language/runtime identified, read the relevant file:

| Language/Runtime     | Reference file                                                                   |
| -------------------- | -------------------------------------------------------------------------------- |
| TypeScript / Node.js | `references/typescript-node.md`                                                  |
| Python               | `references/python.md`                                                           |
| Go                   | `references/go.md`                                                               |
| Rust                 | `references/rust.md`                                                             |
| Ruby                 | `references/ruby.md`                                                             |
| Java / Kotlin (JVM)  | `references/jvm.md`                                                              |
| PHP                  | `references/php.md`                                                              |
| Other / Mixed        | Use `references/typescript-node.md` as the structural template, adapt tool names |

If the user's language isn't in this list, use the TypeScript reference as a
structural guide and reason about equivalents yourself, explaining your choices.

---

## Phase 2: Summarize and confirm

Before generating any files, present a brief summary of what you're about to
build and ask for confirmation:

```
Here's what I'll scaffold:

**Language:** Python 3.12
**Package manager:** uv
**Task runner:** Taskfile (task)
**Linter:** Ruff
**Formatter:** Ruff format
**Tests:** pytest + pytest-cov
**Commit hooks:** pre-commit
**Commit format:** Conventional Commits (commitizen)

**Docs structure:**
- docs/adr/   Architecture Decision Records
- docs/chats/ Session summaries
- docs/plans/ Plans and todos
- docs/specs/ Feature specifications

Does this look right, or would you change anything?
```

Wait for confirmation before proceeding.

---

## Phase 3: Generate the scaffold

Generate files in this order — it makes the most sense narratively and ensures
CLAUDE.md is the first thing written (since it references everything else):

1. `CLAUDE.md`
2. Task runner config
3. Commit hook config + commitlint/commitizen config
4. Lint + format tool configs
5. Test tool config (if needed)
6. `docs/adr/0000-template.md`
7. `docs/adr/0001-initial-toolchain-choices.md` (document the scaffold decisions)
8. `docs/chats/YYYY-MM-DD-project-bootstrap.md` (summary of this session)
9. `docs/plans/README.md`
10. `docs/specs/README.md`
11. `README.md`
12. `.gitignore`
13. Any language-specific files (package.json, pyproject.toml, go.mod, etc.)

### CLAUDE.md generation guidelines

CLAUDE.md is the most important file. It should feel customized, not templated.
Use the developer's actual tool names, not placeholders. Sections to always include:

- **Project Overview** — pre-filled with what they told you, marked [CUSTOMIZE] for gaps
- **How We Work Together** — the docs-as-context loop (this is the same across all stacks)
- **Documentation Rules** — the four doc types and when to use each
- **Commit Workflow** — using their actual task runner command and commit format
- **Task Reference** — all the tasks with their actual command names
- **Code Conventions** — language-specific defaults from the reference file
- **Customization Checklist** — a to-do list for the developer to finish setup

### Taskfile conventions

Always use Taskfile (`Taskfile.yml`) as the task runner regardless of language,
unless the developer has a strong preference for something else (Makefile is an
acceptable alternative). Taskfile is language-agnostic and provides a consistent
interface across stacks.

Standard tasks to always include:

```
task check        # lint + test + docs:check
task lint         # linter
task lint:fix     # auto-fix
task test         # test suite
task test:watch   # watch mode (if the framework supports it)
task docs:check   # verify docs updated when src changed
task commit       # check + commit with MSG variable
task commit:quick # stage all + commit
task adr          # scaffold new ADR
task spec         # scaffold new spec
task summary      # print the chat summary prompt
```

Adapt the underlying commands to the language. The task names stay the same —
this is what makes the workflow portable.

### docs:check hook logic

The docs:check script is language-agnostic — it just looks at git staged files.
The same shell logic works everywhere:

```sh
STAGED_SRC=$(git diff --cached --name-only | grep -E '^src/' || true)
STAGED_DOCS=$(git diff --cached --name-only | grep -E '^docs/' || true)
if [ -n "$STAGED_SRC" ] && [ -z "$STAGED_DOCS" ]; then
  echo "✗ DOCS CHECK FAILED — source files staged but no docs updated"
  exit 1
fi
```

Adjust the `src/` pattern to match the language's source directory convention
(e.g., `lib/` for Ruby, the module name for Go, `src/` for most others).

---

## Phase 4: After generation

Once all files are created:

1. **Print the setup steps** — the exact commands to run to initialize the repo,
   install dependencies, and make the first commit. Use their actual tool names.

2. **Remind them of the customization checklist** — point them to the checklist
   at the bottom of CLAUDE.md.

3. **Offer next steps** — ask if they'd like help with:
   - Writing their first real ADR (`0002-...`)
   - Setting up CI (GitHub Actions or equivalent)
   - Writing a spec for their first feature
   - Anything else specific to their stack

---

## Quality standards

- Never generate placeholder tool names — always use the real ones for their stack
- Every task in Taskfile.yml must work with the tools being configured
- CLAUDE.md must reference the actual task names (not generic names)
- The first ADR must document the actual decisions made in this session
- The chat summary must reflect this actual conversation, not be a generic template
- If the developer mentioned anything specific (a domain, a product name, a team),
  include it in the Project Overview section of CLAUDE.md
- `.gitignore` must be appropriate for their language and tools

---

## Reference files

Read the relevant language reference before suggesting tools or generating configs:

- `references/typescript-node.md` — TS/JS/Node.js defaults and configs
- `references/python.md` — Python defaults and configs
- `references/go.md` — Go defaults and configs
- `references/rust.md` — Rust defaults and configs
- `references/ruby.md` — Ruby defaults and configs
- `references/jvm.md` — Java/Kotlin defaults and configs
- `references/php.md` — PHP defaults and configs
