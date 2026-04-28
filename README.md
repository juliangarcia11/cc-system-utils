# Claude Code System Utilities

Personal utilities for working with [Claude Code](https://claude.ai/claude-code). Not officially supported — use as starting points or inspiration.

## Contents

### Skills

Skills are registered with Claude Code and invoked as slash commands (e.g. `/repo-scaffold`).

| Skill                                          | What it does                                                                                                                                                       |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`repo-scaffold-skill/`](repo-scaffold-skill/) | Interviews you about your stack, then scaffolds a complete starter repo: CLAUDE.md, docs structure, lint/test/format/commit-hook configs, Taskfile, and .gitignore |
| [`refine-dev-loop.md`](refine-dev-loop.md)     | Assesses and updates your development loop documentation across projects; seeds a default if none exists                                                           |

### Prompts

Drop-in prompts — paste into a session or reference them directly. No registration needed.

| Prompt                                       | What it does                                                                               |
| -------------------------------------------- | ------------------------------------------------------------------------------------------ |
| [`extract-dev-loop.md`](extract-dev-loop.md) | Extracts the current session's dev workflow into a `WORKFLOW.md` artifact for human review |

## Registering a skill

Skills live in a directory with a `SKILL.md` file. To register one, add it to `~/.claude/settings.json` (global) or `.claude/settings.json` (project-scoped):

```json
{
  "skills": [
    {
      "name": "repo-scaffold",
      "path": "/absolute/path/to/cc-system-utils/repo-scaffold-skill"
    }
  ]
}
```

The `name` becomes the slash command. The `path` must point to the directory containing `SKILL.md`.

**To let Claude Code do it for you**, open a session and say:

> I cloned `cc-system-utils` to `/path/to/cc-system-utils`. Register the `repo-scaffold-skill` in my global Claude Code settings as `/repo-scaffold`.

## Author

- Me: [@juliangarcia11](https://github.com/juliangarcia11)
- Claude: [@claude](https://claude.ai/claude)
