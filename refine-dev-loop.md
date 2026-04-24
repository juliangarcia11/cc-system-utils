Create/refine your in-memory Developement Loop definition from your projects and global preferences.

When /refine-dev-loop is invoked, follow these steps exactly. Do not skip steps or combine them out of order.

---

## Step 1 — Assess memory across all projects

Use Glob to find all files matching `~/.claude/projects/*/memory/*.md`. Read every file whose name starts with `feedback_` or `user_`. Count the total number found across all projects.

**Define "sparse"**: fewer than 3 such files found in total.

---

## Step 2 — Read existing global CLAUDE.md

Read `~/.claude/CLAUDE.md` if it exists. Note its full contents. If it does not exist, note that it is absent.

---

## Step 3 — Read current project CLAUDE.md

Read `CLAUDE.md` in the current working directory if it exists. Note the docs folder name used (default: `docs/`), any existing dev loop steps, and any project-specific conventions already documented.

---

## Step 4 — Seed default loop into memory if sparse

If memory is sparse, write the following to the current project's memory directory as `feedback_dev_loop.md` (use the memory path shown in the system context):

```
---
name: Default development loop
description: Research → Plan → Implement → Validate → Document loop used when no project-specific loop is defined
type: feedback
---

The default development loop is:

1. **Research** — read the codebase before doing anything so Claude understands what exists
2. **Plan** — define exactly what to build and get user sign-off before writing any code
3. **Implement** — phase by phase with verification at each step
4. **Validate** — confirm the result matches the plan before moving on
5. **Document** — update chats, ADRs, plans, and usage docs in the project's docs folder; docs belong in version control but must be excluded from build artifacts

**Why:** The user wants to maintain knowledge ownership and drive the process. Surfacing the plan before each phase gives them a chance to redirect rather than reviewing a fait accompli.

**How to apply:** At the start of any phase or significant work item, always surface the plan first and wait for explicit confirmation before writing code.
```

Also update `MEMORY.md` in that memory directory to include a pointer to `feedback_dev_loop.md`.

---

## Step 5 — Detect build config and docs exclusion

In the current working directory, check for the presence of: `vite.config.*`, `webpack.config.*`, `tsconfig.json`, `next.config.*`, `astro.config.*`, and the `build` script in `package.json`.

Determine the appropriate way to exclude the docs folder from build output for this stack:

- **Vite**: add `docs/` to the `build.rollupOptions.external` or confirm it is outside `src/`
- **Next.js**: confirm docs folder is not under `app/` or `pages/`; note it in `next.config` if needed
- **tsc**: confirm `docs/` is listed under `exclude` in `tsconfig.json`
- **webpack**: confirm docs folder is not included in the entry graph
- **Unknown**: note that the developer should manually verify docs are excluded from build output

Note the finding — you will include it in the per-project draft.

---

## Step 6 — Write global draft to `~/.claude/CLAUDE.draft.md`

Synthesize everything gathered (memory files from all projects + existing global CLAUDE.md) into a proposed new global CLAUDE.md. Write it to `~/.claude/CLAUDE.draft.md`.

The global file should cover:

- The dev loop phases (Research → Plan → Implement → Validate → Document) and what each means
- Communication style and confirmation requirements extracted from memory
- Any other universal working preferences found in memory

Do NOT include project-specific items (stack details, folder names, commands) in this file. Keep it about style, flow, and universal expectations.

Format as clean markdown suitable for use as a CLAUDE.md instruction file.

---

## Step 7 — Always write a per-project `CLAUDE.draft.md`

Write `CLAUDE.draft.md` in the current working directory. This file is always generated, whether memory was sparse or rich.

Structure it as follows:

```markdown
# [Project Name] — Dev Loop Overrides

> This file captures project-specific additions to the global dev loop defined in ~/.claude/CLAUDE.md.
> Review, edit, then rename to CLAUDE.md (merging with any existing content).

## Docs Folder

- Docs folder: `docs/` (change this if the project uses a different name)
- Build exclusion: [insert finding from Step 5]

## Loop Additions or Overrides

<!-- Add any project-specific steps, phase names, or deviations from the global loop here. -->
<!-- Examples: "Run pnpm test after every Implement phase", "ADRs go in docs/adr/", "Always reindex after vault changes" -->

## Project-Specific Confirmation Requirements

<!-- Any confirmations beyond the global defaults — e.g., "always confirm before touching the DB schema" -->
```

Populate each section with anything relevant found in the current project's CLAUDE.md or memory. Leave placeholders where nothing was found.

---

## Step 8 — Report to the user

Tell the user:

1. How many memory files were found across all projects and whether the default loop was seeded
2. What build config was detected and what the docs exclusion situation is
3. That two draft files have been written:
   - `~/.claude/CLAUDE.draft.md` — global style and loop
   - `CLAUDE.draft.md` — project-specific overrides
4. Next steps: review and edit both files, then rename each to `CLAUDE.md` (merging the project draft with any existing project `CLAUDE.md`)

Keep the report concise — one short paragraph per item above.
